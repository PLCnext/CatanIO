## Node-RED Setup on Catan

This example shows how to run Node-RED on a Catan device with access to USB and RS485 interfaces.

Two deployment options are available:

- Podman Compose
- systemd service (rootless Podman)

## Option 1: Podman Compose

### Service Overview

- Uses the official `nodered/node-red:latest` image
- Automatically restarts unless stopped manually
- Exposes the Node-RED web interface on port `1880`
- Stores persistent data in `./nodered`

### Compose File

```yaml
services:
  node-red:
    image: docker.io/nodered/node-red:latest
    restart: unless-stopped
    environment:
      - TZ=Europe/Amsterdam
    ports:
      - "1880:1880"
    volumes:
      - ~/nodered:/data
# USB serial
      - /dev/usb-devices:/dev/usb-devices
    devices:
# RS485(2)     
      - /dev/ttymxc2:/dev/ttymxc2
# RS485(1)      
      - /dev/ttymxc3:/dev/ttymxc3

# Permissions for the admin user
    userns_mode: keep-id
    user: 1002:1002
```

### Create directory

```bash
mkdir ~/nodered
```

### Start

```bash
podman compose up -d
```

### Stop

```bash
podman compose down
```

## Option 2: systemd Service (Rootless Podman)

This variant runs Node-RED as a native systemd user service.

### Service File

Create:

```text
~/.config/systemd/user/nodered.service
```

```ini
[Unit]
Description=Node-RED rootless Podman container
After=network-online.target
Wants=network-online.target

[Service]
Restart=always
RestartSec=10
ExecStartPre=/usr/bin/mkdir -p ~/nodered
ExecStartPre=-/usr/bin/podman rm -f nodered

ExecStart=/usr/bin/podman run \
  --name nodered \
  --rm \
  --userns=keep-id \
  --user 1002:1002 \
  --env TZ=Europe/Amsterdam \
  --publish 1880:1880 \
  --volume ~/nodered:/data \
  --device /dev/ttymxc2:/dev/ttymxc2 \
  --device /dev/ttymxc3:/dev/ttymxc3 \
  docker.io/nodered/node-red:latest

ExecStop=/usr/bin/podman stop nodered
ExecStopPost=-/usr/bin/podman rm -f nodered

TimeoutStartSec=120
TimeoutStopSec=30

[Install]
WantedBy=default.target
```

### Enable and Start

Reload systemd:

```bash
systemctl --user daemon-reload
```

Enable automatic startup:

```bash
systemctl --user enable nodered.service
```

Start the service:

```bash
systemctl --user start nodered.service
```

Check status:

```bash
systemctl --user status nodered.service
```

View logs:

```bash
journalctl --user -u nodered.service -f
```

Stop Node-RED:

```bash
systemctl --user stop nodered.service
```

## Node-RED dependencies

The following Node-RED addons are required:

- CoAP addon:
  https://flows.nodered.org/node/node-red-contrib-coap

- Protobuf addon:
  https://flows.nodered.org/node/@bveenema/node-red-protobuf

Optional (for KNX usage):

- KNX addon:
  https://flows.nodered.org/node/node-red-contrib-knx-ultimate

  Requires a KNX interface configured on the controller IP address.

## Interfaces

### USB Serial

- All USB serial devices are exposed via `/dev/usb-devices`
- Allows integration of gateways such as:
  - M-Bus
  - wM-Bus
  - EnOcean
  - Modbus RTU USB adapters

### RS485

- `/dev/ttymxc3` → RS485(1)
- `/dev/ttymxc2` → RS485(2)

Typical use cases:

- Modbus RTU
- M-Bus gateways
- Proprietary RS485 devices


## Usage

Open Node-RED:

```text
http://<device-ip>:1880
```

## Protobuf Configuration

- The required protobuf files for Catan must be copied into Node-RED.
- In the provided example, all protobuf definitions are merged into a single file (`ui.proto`).
- This file is used for encoding and decoding CoAP payloads exchanged with the controller.

## Example Flow

An example Node-RED flow is provided and can be imported directly into the Node-RED editor.

The example demonstrates:

- Configuration of Catan I/Os
- Reading digital inputs
- Reading analog inputs
- Writing digital outputs
- CoAP communication
- Protobuf encoding/decoding