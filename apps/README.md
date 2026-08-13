# App development

An application includes an OCI container that is can interact with the system through a defined set of interfaces.
Within this environment, the container access and use selected system services and resources, provided that the appropriate interfaces are enabled.

The following interfaces can be utilized by an OCI container as part of the application. In order to enable their usage, the corresponding settings should be configured accordingly.

For more information see: 
* https://github.com/PLCnext/PLCnextAppExamples
* https://github.com/PLCnext/App-Info-Schema

## Podman, Quadlet, and Compose recommendations for Catan C1

### Permissions

All interfaces have permissions for user `admin` and group `plcnext` for individual use and `app_user` for PLCnext Store apps. 

Necessary to pass host system permissions to the container.

For compose: 

```yaml
userns_mode: keep-id
user: 1002:1002
```

- The `admin` user has UID `1002`
- The `plcnext` group has GID `1002`
- `keep-id` ensures correct user mapping between the host and the container

**Important for PLCnext apps:**
- The `app_user` must use UID `1001`
- The group (`plcnext`) remains GID `1002`

For Quadlet: 
```yaml
User=1002:1002
UserNS=keep-id
```

### Network

Necessary to access resources on the host system. This includes the KNX interface and the CodeMeter Server license system.

* Network=host

Only for CodeMeter Server

* PodmanArgs=--pid host

### RS-485

For accessing to the two RS-485 interfaces.

RS-485(1):
* AddDevice=/dev/ttymxc3

RS-485(2):
* AddDevice=/dev/ttymxc2

### USB interface

USB serial devices are added to:

* /dev/usb-devices

## System Ressources Catan C1

### CodeMeter Server

Available

* Port 22350
* Bind to any ip 

### KNX Interface

The KNX interface is accessible locally on the host system. External access to the KNX interface can be configured through the firewall. 

By default, reachable via LAN1 and LAN2

* Port 3671 
* Bind to any IP address
