## Tailscale Compose Setup (Rootless)

This Compose file runs a Tailscale client using rootless Podman mode on a Catan device.

### Service Overview

- Runs the Tailscale client to connect the container securely to a private tailnet
- Uses Tailscale userspace networking without requiring `/dev/net/tun` or the `NET_ADMIN` capability
- Provides a local SOCKS5 proxy for accessing devices and services within the tailnet
- Stores persistent Tailscale state in a local directory

### Prerequisites

Create the persistent state directory before starting the container:

```bash
mkdir -p /home/admin/tailscale
chmod 700 /home/admin/tailscale
```


### Environment File

Create a `.env` file next to the Compose file:

```dotenv
TS_AUTHKEY=tskey-auth-xxxxxxxxxxxxxxxx
TS_HOSTNAME=catan-tailscale
```

Restrict access to the file because it contains the Tailscale auth key:

```bash
chmod 600 .env
```

Do not commit the `.env` file to a Git repository.

### Compose File

Create a file named `compose.yaml`:

```yaml
services:
  tailscale:
    image: docker.io/tailscale/tailscale:latest
    container_name: tailscale
    hostname: ${TS_HOSTNAME:-catan-tailscale}
    restart: unless-stopped

    userns_mode: keep-id
    user: 1002:1002 

    # Use the network namespace of the Catan host.
    # The tailscale0 interface is created directly on the host.
    network_mode: host

    environment:
      # Authentication and node configuration
      TS_AUTHKEY: ${TS_AUTHKEY}
      TS_HOSTNAME: ${TS_HOSTNAME:-catan-tailscale}
      TS_AUTH_ONCE: "true"

      # Store the node identity persistently
      TS_STATE_DIR: /var/lib/tailscale

      # Use kernel networking with a real TUN interface
      TS_USERSPACE: "true"

      # Do not modify the DNS configuration of the Catan host
      TS_ACCEPT_DNS: "false"

    volumes:
      - /home/admin/tailscale:/var/lib/tailscale
```

### Usage

1. Start the service:

   ```bash
   podman compose up -d
   ```

2. View the logs:

   ```bash
   podman compose logs -f
   ```

3. Check the Tailscale status:

   ```bash
   podman exec tailscale tailscale status
   ```

4. Display the assigned Tailscale IP addresses:

   ```bash
   podman exec tailscale tailscale ip
   ```

5. Test access to another tailnet device from inside the container:

   ```bash
   podman exec tailscale tailscale ping <tailscale-hostname-or-ip>
   ```

6. Stop the service:

   ```bash
   podman compose down
   ```


### Notes

- Rootless Podman requires the user namespace to be configured with `keep-id`
- The UID and GID in the `user` entry must match the rootless Podman user on the Catan device
- The Tailscale auth key is provided through the `TS_AUTHKEY` environment variable
- `TS_AUTH_ONCE=true` prevents unnecessary reauthentication when persistent state already exists
- Persistent Tailscale state is stored in `/home/admin/tailscale`
- The persistent directory must exist before the container is started
- For production deployments, pin the container image to a tested version instead of using `latest`
- Setup keys are sensitive credentials and must not be stored in Git repositories or published in logs