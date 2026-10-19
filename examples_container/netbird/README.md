## NetBird Compose Setup (Rootless)

This Compose file runs a NetBird client using rootless Podman mode on a Catan device.

### Service Overview

- Runs the NetBird client to connect the Catan device securely to a private NetBird network
- Uses the network namespace of the Catan host
- Creates the NetBird network interface directly in the host network namespace
- Allows services running on the Catan device to be reached through the NetBird network
- Stores persistent NetBird configuration and peer identity in a local directory

### Prerequisites

Create the persistent state directory before starting the container:

```bash
mkdir -p /home/admin/netbird
chmod 700 /home/admin/netbird
```

### Environment File

Create a `.env` file next to the Compose file:

```dotenv
NB_SETUP_KEY=<your-setup-key>
NB_HOSTNAME=catan-netbird
```

Restrict access to the file because it contains the NetBird setup key:

```bash
chmod 600 .env
```

Do not commit the `.env` file to a Git repository.

### Compose File

Create a file named `compose.yaml`:

```yaml
services:
  netbird:
    image: docker.io/netbirdio/netbird:rootless-latest
    container_name: netbird
    hostname: ${NB_HOSTNAME:-catan-netbird}
    restart: unless-stopped

    # Use the network namespace of the Catan host.
    # The NetBird interface is created directly on the host.
    network_mode: host

    environment:
      # Authentication
      NB_SETUP_KEY: ${NB_SETUP_KEY}

    volumes:
      # Store the peer identity and configuration persistently
      - /home/admin/netbird:/var/lib/netbird

    devices:
      # Provide access to the Linux TUN device
      - /dev/net/tun:/dev/net/tun

    userns_mode: keep-id
    user: 1002:1002 
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

3. Check the NetBird status:

   ```bash
   podman exec netbird netbird status
   ```

4. Display detailed NetBird connection information:

   ```bash
   podman exec netbird netbird status --detail
   ```

5. Display the network interfaces on the Catan host:

   ```bash
   ip address
   ```

6. Test access to another NetBird peer:

   ```bash
   ping <netbird-peer-ip>
   ```

7. Stop the service:

   ```bash
   podman compose down
   ```

### Notes

- Container root is mapped to the unprivileged `admin` user on the host
- The container uses the host network namespace to provide transparent access to Catan services
- The NetBird setup key is provided through the `NB_SETUP_KEY` environment variable
- The persistent directory must exist before the container is started
- Services must listen on the NetBird IP address or on all network interfaces to be reachable through NetBird
- The Catan firewall must allow the required incoming connections through the NetBird interface
- For production deployments, pin the container image to a tested version instead of using `latest`
- Setup keys are sensitive credentials and must not be stored in Git repositories or published in logs