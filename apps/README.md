# App development

An application include an OCI container that is able to interact with the system through a defined set of interfaces.
Within this environment, the container can access and make use of selected system services and resources, provided that the appropriate interfaces are enabled.

The following interfaces can be utilized by an OCI container as part of the application. In order to enable their usage, the corresponding settings should be configured accordingly.

For more information see: 
* https://github.com/PLCnext/PLCnextAppExamples
* https://github.com/PLCnext/App-Info-Schema

## Podman Quadlet and compose recommendation for Catan C1

### Permissions

All interface have permissions for user `admin` and group `plcnext` for individuell use and `app_user` for PLCnext Store apps. 

Necessary to pass system permissions to the container.

For compose: 

```yaml
userns_mode: keep-id
user: 1002:1002
```

- The `admin` user has UID `1002`
- The `plcnext` group has GID `1002`
- `keep-id` ensures correct mapping between host and container users

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

Only for Codemeter Server

* PodmanArgs=--pid host

### RS485

For accessing the two RS485 interfaces.

RS485(1):
* AddDevice=/dev/ttymxc3

RS485(2):
* AddDevice=/dev/ttymxc2

### USB interface

USB serial devices are added to:

* /dev/usb-devices

## System ressources Catan C1

### Codemeter Server

Available

* Port 22350
* Bind to any ip 

### KNX Interface

The KNX interface is accessible locally on the host. The accessibility of the KNX interface from outside can be configured through the firewall. 

Default reachable on switch LAN1+LAN2

* Port 3671 
* Bind to any ip




