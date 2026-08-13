## Home Assistant Compose Setup

This Compose file runs a Home Assistant instance on a Catan C1 and provides access to its USB and RS-485 interfaces.

### Service Overview
- Uses the official Home Assistant container image
- Automatically restarts unless stopped manually
- Exposes the Home Assistant web interface (default port `8123`)
- Stores persistent configuration and data in `~/config`

### Compose File
```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: "ghcr.io/home-assistant/home-assistant:stable"
    volumes:
      - ~/config:/config
      - /etc/localtime:/etc/localtime:ro
      - /run/dbus:/run/dbus:ro
      # USB serial devices
      - /dev/usb-devices:/dev/usb-devices
    devices:
      # RS485(1) and RS485(2)
      - /dev/ttymxc2:/dev/ttymxc2
      - /dev/ttymxc3:/dev/ttymxc3
    restart: unless-stopped
    network_mode: host
    # Permissions for the admin user
    userns_mode: keep-id
    user: 1002:1002
    environment:
      TZ: Europe/Amsterdam
```

### Interfaces
- **USB Serial**
  - All USB serial devices are available through `/dev/usb-devices`
  - Enables the use of gateways such as M-Bus, wM-Bus, EnOcean.

- **RS485**
  - `/dev/ttymxc3` → RS-485 (1)
  - `/dev/ttymxc2` → RS-485 (2)

### Usage
1. Start the stack:
   ```bash
   podman compose up -d
   ```
2. Open Home Assistant in a web browser:
   - http://<device-ip>:8123

3. Stop the stack:
   ```bash
   podman compose down
   ```

### Protobuf Configuration
- The required protobuf files for Catan must be provided to the Home Assistant environment
- Typically handled via external services or scripts
- In the provided example setup, all protobuf definitions are merged into a single file (`ui.proto`) for simplified usage

### Notes
- All mounted directories must exist on the host system before starting the container
- Podman does not create missing mount paths automatically
- Ensure correct device permissions for accessing serial and fieldbus interfaces

### Example Use Case
A sample setup can be used to:

- Configure Catan I/Os
- Read digital and analog inputs
- Control outputs (DO)
- Integrate KNX or serial-based fieldbus devices

This setup can be extended using Home Assistant automations, scripts, and dashboards.
