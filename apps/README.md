# App development

An application include an OCI container that is able to interact with the system through a defined set of interfaces.
Within this environment, the container can access and make use of selected system services and resources, provided that the appropriate interfaces are enabled.

The following interfaces can be utilized by an OCI container as part of the application. In order to enable their usage, the corresponding settings should be configured accordingly.

For more information see: https://github.com/PLCnext/PLCnextAppExamples

## Podman Quadlet recommendation for Catan C1

### Firmware requirements

Necessary to pass system permissions to the container. The app in the container has to run without root rights.

* GroupAdd=keep-groups
* UserNS=keep-id

### Network

Necessary to access resources on the host system. This includes the KNX interface and the CodeMeter Server license system.

* Network=host

Only for Codemeter Server

* PodmanArgs=--pid host

### RS485

For accessing the two RS485 interfaces.

* AddDevice=/dev/ttymxc2
* AddDevice=/dev/ttymxc3

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




