## NetBird Compose Setup (Rootless)

This Compose file runs a NetBird client using rootless Podman mode on a Catan device.

### Service Overview
- Runs the NetBird client to connect the device securely to a private network
- Stores persistent configuration data in a local directory

### Compose File
```yaml
services:
  netbird:
    image: docker.io/netbirdio/netbird:latest
    container_name: netbird
    restart: unless-stopped
    volumes:
      - ./netbird:/etc/netbird
    network_mode: host
    environment:
      - NB_SETUP_KEY=<your-setup-key>
    # Permissions for the admin user
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

3. Stop the service:
   ```bash
   podman compose down
   ```

### Notes
- Rootless Podman requires the user namespace to be configured with `keep-id`
- The NetBird setup key must be provided through the environment variable (`NB_SETUP_KEY`)
- Persistent configuration is stored in `./netbird`
- All mount paths must be created manually in the host filesystem. Podman does not create them automatically
- Depending on the host configuration, additional permissions or capabilities may be required for networking

