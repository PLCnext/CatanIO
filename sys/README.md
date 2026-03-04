# System API – Protobuf Documentation

This documentation describes the **system messages (sys/\*)** of the CatanIO platform.  
The interface is based on **Protocol Buffers** and exposed via **CoAP**.

---

## General Architecture

- **Transport**: CoAP
- **Serialization**: protobuf (proto3)
- **Pattern**: Request / Response
- **Central container**: `SysMessage`
- **Payload selection**: `oneof payload` depending on `msg_type`

## Notes

- **Setting types** allow `DONT_CHANGE`, `DEFAULT`, or explicit values
- **oneof payload** guarantees exactly **one payload per message**
- Fields marked *do not reuse* are intentionally reserved and must not be reassigned

## SysMessage – Central Message Container

`SysMessage` is the wrapper for **all system‑level operations**.  
The operation type is defined by `msg_type`; the corresponding payload is provided via `oneof payload`.

---

## Endpoint overview

## System API – Endpoint Overview

|Method|Endpoint|Description|
|---|---|---|
|GET|`coap://<ip>:<port>/sys/cfg`|Read system configuration|
|PATCH|`coap://<ip>:<port>/sys/cfg`|Write system configuration|
|POST|`coap://<ip>:<port>/sys/discovery`|Start device discovery|
|POST|`coap://<ip>:<port>/sys/discovery`|Control discovery process|
|POST|`coap://<ip>:<port>/sys/audio`|Play system or alarm sounds|
|GET|`coap://<ip>:<port>/sys/signal`|Read system signal status|
|GET|`coap://<ip>:<port>/sys/load`|Read system load information|
|GET|`coap://<ip>:<port>/sys/ip`|Read TCP/IP configuration|
|PATCH|`coap://<ip>:<port>/sys/ip`|Write TCP/IP configuration|
|GET|`coap://<ip>:<port>/sys/time`|Read system time|
|PATCH|`coap://<ip>:<port>/sys/time`|Set system time|
|GET|`coap://<ip>:<port>/sys/info`|Read device information|
|GET|`coap://<ip>:<port>/sys/status`|Read system status|
|GET|`coap://<ip>:<port>/sys/ui/cal`|Read UI calibration data|
|PATCH|`coap://<ip>:<port>/sys/ui/cal`|Write UI calibration data|
|POST|`coap://<ip>:<port>/sys/reboot`|Reboot system|
|GET|`coap://<ip>:<port>/sys/neighbors`|Read neighbor information|
|GET|`coap://<ip>:<port>/sys/dt4i`|Read DT4I configuration|
|PATCH|`coap://<ip>:<port>/sys/dt4i`|Write DT4I configuration|
|PATCH|`coap://<ip>:<port>/sys/display/connection`|Set display connection status|
|GET|`coap://<ip>:<port>/sys/fw`|Read firmware update information|
|POST|`coap://<ip>:<port>/sys/memory`|Execute memory command|
|POST|`coap://<ip>:<port>/sys/identify`|Trigger device identification|
|POST|`coap://<ip>:<port>/sys/led`|Control LEDs|

---

### Structure

|Field|Type|Tag|Description|
|---|---|---|---|
|msg_type|MsgType|1|Defines the type of message|
|payload|oneof|10..23|Contains the concrete payload|

---

## MsgType – Message Types

|Name|Value|Payload|
|---|---|---|
|TYPE_NONE|0|ReqRespEmpty|
|TYPE_REQ_READ_INFO|1|ReqRespEmpty|
|TYPE_RESP_READ_INFO|2|ReqRespInfo|
|TYPE_REQ_READ_STATUS|3|ReqRespEmpty|
|TYPE_RESP_READ_STATUS|4|RespStatus|
|TYPE_REQ_WRITE_INFO|5|ReqRespInfo|
|TYPE_REQ_READ_UI_CAL|6|ReqRespEmpty|
|TYPE_REQ_WRITE_UI_CAL|7|ReqRespUiCal|
|TYPE_RESP_READ_UI_CAL|8|ReqRespUiCal|
|TYPE_REQ_REBOOT|9|ReqRespEmpty/RequBlCmd|
|TYPE_REQ_NEIGHBORS|10|ReqRespEmpty|
|TYPE_RESP_NEIGHBORS|11|RespNeighborInfo|
|TYPE_REQ_WRITE_DT4I|12|ReqRespDt4i|
|TYPE_REQ_READ_DT4I|13|ReqRespEmpty|
|TYPE_RESP_READ_DT4I|14|ReqRespDt4i|
|TYPE_RESP_REBOOT|15|ReqRespEmpty|
|TYPE_REQ_WRITE_DISP_CONN_STATUS|19|ReqDispConnStatus|
|TYPE_RESP_WRITE_DISP_CONN_STATUS|20|ReqRespEmpty|
|TYPE_REQ_READ_FW_UPDATE_INFO|21|ReqRespEmpty|
|TYPE_RESP_READ_FW_UPDATE_INFO|22|RespFwUpdateInfo|
|TYPE_REQ_MEMORY_CMD|23|ReqMemoryCmd|
|TYPE_REQ_IDENTIFY|24|ReqIdentify|
|TYPE_REQ_START_DISCOVERY|25|ReqRespEmpty|
|TYPE_REQ_LED_CTRL|26|ReqLedCtrl|
|TYPE_RESP_MEMORY_CMD|27|RespMemoryCmd|

---

## sys/cfg – System Configuration

### Purpose
Read and write **persistent system configuration**, such as device name, RS485 settings, language, and passwords.

---

### SysCfgReq (PATCH)

|Field|Type|Description|
|---|---|---|
|ctrl_connect_failure_timeout_s|Uint32Setting|Timeout for controller connection failure|
|name|String64Setting|Device name|
|rs485_ch1_termination|BooleanSetting|RS485 CH1 termination|
|rs485_ch1_biasing|BooleanSetting|RS485 CH1 biasing|
|rs485_ch2_termination|BooleanSetting|RS485 CH2 termination|
|rs485_ch2_biasing|BooleanSetting|RS485 CH2 biasing|
|password1|Uint64Setting|Password slot 1|
|password2|Uint64Setting|Password slot 2|
|password3|Uint64Setting|Password slot 3|
|language|Language|UI language|
|local_setup_enabled|BooleanSetting|Allow local setup|

---

### SysCfgResp (GET/PATCH)

|Field|Type|Description|
|---|---|---|
|ctrl_connect_failure_timeout_s|uint32|Active timeout|
|name|string|Device name|
|rs485_ch1_termination|bool|Current state|
|rs485_ch1_biasing|bool|Current state|
|rs485_ch2_termination|bool|Current state|
|rs485_ch2_biasing|bool|Current state|
|password1|uint64|Password hash|
|password2|uint64|Password hash|
|password3|uint64|Password hash|
|linkbus_address|uint64|Internal address|
|language|Language|Active language|
|local_setup_enabled|bool|Current state|

---

## sys/discovery – Device Discovery

### Purpose
Automatic discovery of devices on the network.

---

### ReqDiscover

|Field|Type|Description|
|---|---|---|
|option|DiscoverOption|Controls the discovery process|

#### DiscoverOption

|Name|Value|
|---|---|
|OPTION_GET_STATUS|0|
|OPTION_START_READ_ONLY|1|
|OPTION_START_ADDRESS_ALL|2|
|OPTION_GET_RESULT|3|
|OPTION_ABORT|4|

---

### RespDiscover

|Field|Type|Description|
|---|---|---|
|status|StatusCode|Idle or Busy|
|progress|uint32|Progress in percent|
|discover_entries|DiscoverEntry[]|Discovered devices|
|error|ErrorCode|Host error status|

---

## sys/audio – Audio Control

### Purpose
Playback of system and alarm sounds.

---

### ReqAudio

|Field|Type|Description|
|---|---|---|
|action|Action|Audio action|
|sequence|bytes|Sound sequence|

### RespAudio

|Field|Type|Description|
|---|---|---|
|status|Status|Idle or Busy|
|error|ErrorCode|Error state|

---

## sys/ip – Network Configuration

### Purpose
Read and write network adapter configuration.

---

### Adapter

|Field|Type|Description|
|---|---|---|
|id|AdapterId|EN0 or EN1|
|enabled|bool|Adapter enabled|
|can_disable|bool|Can be disabled|
|is_readonly|bool|Read‑only adapter|
|ipv6_enabled|bool|IPv6 enabled|
|can_disable_ipv6|bool|IPv6 configurable|
|ipv4|Ipv4|IPv4 configuration|
|ipv6|Ipv6|IPv6 configuration|

---

### RespTcpIpSettings

|Field|Type|Description|
|---|---|---|
|hostname|string|System hostname|
|adapters|Adapter[]|Network adapters|
|eee|bool|Eco‑mode enabled|

---

### ReqTcpIpSettings

|Field|Type|Description|
|---|---|---|
|adapters|SetAdapter[]|Adapters to modify|

---

## sys/time – System Time

### Purpose
Read and set the local system time.

---

### RespTime

|Field|Type|Description|
|---|---|---|
|local_date_time|uint64|Unix timestamp|

### ReqTime

|Field|Type|Description|
|---|---|---|
|local_date_time|Uint64Setting|New system time|

---

## sys/status – System Status

### RespStatus

|Field|Type|Description|
|---|---|---|
|points_in_failure|uint32|Number of points in FAILURE|
|points_overridden|uint32|Number of overridden points|
|display_connected|bool|Display connected|
|ctrl_connected|bool|Controller connected|
|button_pressed|bool|Button pressed|
|ctrl_connect_failure|bool|Controller connection failure|

---


