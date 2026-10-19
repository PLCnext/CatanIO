## ZeroTier Compose Setup (Rootless)

This Compose file runs a ZeroTier client using rootless Podman mode on a Catan device.

### Service Overview

- Runs the ZeroTier client to connect the Catan device securely to a private ZeroTier network
- Stores persistent ZeroTier configuration and node identity in a local directory

### Prerequisites

Create the persistent state directory before starting the container:

```bash
mkdir -p /home/admin/zerotier
chmod 700 /home/admin/zerotier
```

### Compose File

Create a file named `compose.yaml`:

```yaml
services:
  zerotier:
    image: docker.io/zyclonite/zerotier:latest
    container_name: zerotier
    hostname: catan-zerotier
    restart: unless-stopped

    # Use the network namespace of the Catan host.
    # The ZeroTier interface is created directly on the host.
    network_mode: host

    volumes:
      # Store the node identity and configuration persistently
      - /home/admin/zerotier:/var/lib/zerotier-one

    devices:
      # Provide access to the Linux TUN/TAP device
      - /dev/net/tun:/dev/net/tun
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

3. Check the ZeroTier service status:

   ```bash
   podman exec zerotier zerotier-cli status
   ```

4. Join the configured ZeroTier network:

   ```bash
   podman exec zerotier zerotier-cli join <zerotier-network-id>
   ```

5. Authorize the new member in the ZeroTier Central management interface.

6. Display the joined ZeroTier networks:

   ```bash
   podman exec zerotier zerotier-cli listnetworks
   ```

7. Display the known ZeroTier peers:

   ```bash
   podman exec zerotier zerotier-cli peers
   ```

8. Display the ZeroTier network interface on the Catan host:

   ```bash
   ip address show
   ```

9. Test access to another ZeroTier member:

   ```bash
   ping <zerotier-member-ip>
   ```

10. Leave the ZeroTier network:

    ```bash
    podman exec zerotier zerotier-cli leave <zerotier-network-id>
    ```

11. Stop the service:

    ```bash
    podman compose down
    ```

### Notes

- Podman is executed rootless by the `admin` user
- Container root is mapped to the unprivileged `admin` user on the host
- The container uses the host network namespace to provide transparent access to Catan services
- The persistent directory must exist before the container is started
- The persistent state preserves the ZeroTier node identity across container restarts
- Services must listen on the ZeroTier IP address or on all network interfaces to be reachable through ZeroTier
- The Catan firewall must allow the required incoming connections through the ZeroTier interface
- For production deployments, pin the container image to a tested version instead of using `latest`