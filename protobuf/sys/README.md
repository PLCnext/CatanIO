# CatanIO: System API Documentation

## Overview

This document describes the system-level CoAP endpoints and Protocol Buffers messages of the CatanIO platform.

The System API uses two communication models:

1. Direct CoAP endpoints with dedicated request and response messages
2. Generic system operations transported as `SysMessage` messages via `PATCH /sys`

Operations such as retrieving device information, requesting the system status, calibrating the UI, discovering neighboring devices, retrieving firmware information, rebooting, executing memory-related commands, identifying devices, and controlling LEDs are not implemented as separate CoAP endpoints.
Instead, they are selected through `SysMessage.msg_type` and send via:

```text
PATCH /sys
```

## Dependencies

- `shared.proto`: Shared settings and message types
- `nanopb.proto`: Nanopb field-size and array constraints
- `sys.proto`: System-specific messages, enums, and endpoints

## Direct CoAP endpoint overview

| Method | Endpoint | Request | Response | Description |
|---|---|---|---|---|
| `GET` | `/sys/cfg` | None | `SysCfgResp` | Reads the effective system configuration |
| `PATCH` | `/sys/cfg` | `SysCfgReq` | `SysCfgResp` | Changes the system configuration and returns the effective result |
| `PATCH` | `/sys/discover` | `ReqDiscover` | `RespDiscover` | Queries, starts, retrieves, or aborts device discovery |
| `PATCH` | `/sys/audio` | `ReqAudio` | `RespAudio` | Controls system, identification, button, or alarm audio |
| `GET` | `/sys/signal` | None | `RespSignal` | Reads system signal states |
| `GET` | `/sys/load` | None | `RespLoad` | Reads daemon and station load values |
| `GET` | `/sys/ip` | None | `RespTcpIpSettings` | Reads TCP/IP configuration |
| `PATCH` | `/sys/ip` | `ReqTcpIpSettings` | `RespTcpIpSettings` | Changes TCP/IP configuration and returns the effective result |
| `GET` | `/sys/time` | None | `RespTime` | Reads the system-local Unix timestamp |
| `PATCH` | `/sys/time` | `ReqTime` | `RespTime` | Changes the system-local Unix timestamp |
| `PATCH` | `/sys` | `SysMessage` | `SysMessage` | Executes a generic system operation selected by `msg_type` |

## Generic system operations

Generic system operations are transported through:

```text
PATCH /sys
```

The request and response use the `SysMessage` container.

The requested operation is selected by:

```text
SysMessage.msg_type
```

The associated message is stored in:

```text
SysMessage.payload
```

Only one field of the `payload` oneof may be present in a message.

## `SysMessage`

### Structure

| Field | Number | Type | Description |
|---|---:|---|---|
| `msg_type` | 1 | `MsgType` | Identifies the request or response operation |
| `payload` | Oneof | `oneof` | Contains the payload associated with `msg_type` |

### Payload fields

| Payload field | Number | Message type | Description |
|---|---:|---|---|
| `req_resp_empty` | 10 | `ReqRespEmpty` | Empty request or response payload |
| `req_resp_info` | 11 | `ReqRespInfo` | Device information payload |
| `resp_status` | 12 | `RespStatus` | System status response |
| `req_resp_ui_cal` | 13 | `ReqRespUiCal` | UI calibration payload |
| `resp_neighbor_info` | 14 | `RespNeighborInfo` | Physical neighbor information |
| `req_resp_dt4i` | 15 | `ReqRespDt4i` | DT4I configuration payload |
| `req_disp_conn_status` | 17 | `ReqDispConnStatus` | Display-connection state request |
| `req_bl_cmd` | 18 | `RequBlCmd` | Bootloader command used with a reboot request |
| `resp_fw_update_info` | 19 | `RespFwUpdateInfo` | Firmware update information |
| `req_mem_cmd` | 20 | `ReqMemoryCmd` | Memory command request |
| `req_led_ctrl` | 21 | `ReqLedCtrl` | LED control request |
| `req_identify` | 22 | `ReqIdentify` | Device identification request |
| `resp_mem_cmd` | 23 | `RespMemoryCmd` | Memory command response |

Payload field number 16 is reserved and must not be reused.

### Empty payload

`ReqRespEmpty` does not contain any fields.

It is used for operations that do not require an operation-specific payload.

```protobuf
message ReqRespEmpty {
}
```

## `MsgType`

| Name | Value | Expected payload |
|---|---:|---|
| `TYPE_NONE` | 0 | `req_resp_empty` |
| `TYPE_REQ_READ_INFO` | 1 | `req_resp_empty` |
| `TYPE_RESP_READ_INFO` | 2 | `req_resp_info` |
| `TYPE_REQ_READ_STATUS` | 3 | `req_resp_empty` |
| `TYPE_RESP_READ_STATUS` | 4 | `resp_status` |
| `TYPE_REQ_WRITE_INFO` | 5 | `req_resp_info` |
| `TYPE_REQ_READ_UI_CAL` | 6 | `req_resp_empty` |
| `TYPE_REQ_WRITE_UI_CAL` | 7 | `req_resp_ui_cal` |
| `TYPE_RESP_READ_UI_CAL` | 8 | `req_resp_ui_cal` |
| `TYPE_REQ_REBOOT` | 9 | `req_resp_empty` or `req_bl_cmd` |
| `TYPE_REQ_NEIGHBORS` | 10 | `req_resp_empty` |
| `TYPE_RESP_NEIGHBORS` | 11 | `resp_neighbor_info` |
| `TYPE_REQ_WRITE_DT4I` | 12 | `req_resp_dt4i` |
| `TYPE_REQ_READ_DT4I` | 13 | `req_resp_empty` |
| `TYPE_RESP_READ_DT4I` | 14 | `req_resp_dt4i` |
| `TYPE_RESP_REBOOT` | 15 | `req_resp_empty` |
| `TYPE_REQ_WRITE_DISP_CONN_STATUS` | 19 | `req_disp_conn_status` |
| `TYPE_RESP_WRITE_DISP_CONN_STATUS` | 20 | `req_resp_empty` |
| `TYPE_REQ_READ_FW_UPDATE_INFO` | 21 | `req_resp_empty` |
| `TYPE_RESP_READ_FW_UPDATE_INFO` | 22 | `resp_fw_update_info` |
| `TYPE_REQ_MEMORY_CMD` | 23 | `req_mem_cmd` |
| `TYPE_REQ_IDENTIFY` | 24 | `req_identify` |
| `TYPE_REQ_START_DISCOVERY` | 25 | `req_resp_empty` |
| `TYPE_REQ_LED_CTRL` | 26 | `req_led_ctrl` |
| `TYPE_RESP_MEMORY_CMD` | 27 | `resp_mem_cmd` |

Message type values 16, 17, and 18 are reserved and must not be reused.

### Reboot payload selection

`TYPE_REQ_REBOOT` supports two payload variants:

- `req_resp_empty` for a normal reboot
- `req_bl_cmd` for a reboot combined with a bootloader command

The selected oneof field determines which reboot variant is requested.

## Message and operation mapping

| Operation | Request type | Request payload | Response type | Response payload |
|---|---|---|---|---|
| Read device information | `TYPE_REQ_READ_INFO` | `req_resp_empty` | `TYPE_RESP_READ_INFO` | `req_resp_info` |
| Read system status | `TYPE_REQ_READ_STATUS` | `req_resp_empty` | `TYPE_RESP_READ_STATUS` | `resp_status` |
| Write device information | `TYPE_REQ_WRITE_INFO` | `req_resp_info` | Implementation-specific | Implementation-specific |
| Read UI calibration | `TYPE_REQ_READ_UI_CAL` | `req_resp_empty` | `TYPE_RESP_READ_UI_CAL` | `req_resp_ui_cal` |
| Write UI calibration | `TYPE_REQ_WRITE_UI_CAL` | `req_resp_ui_cal` | Implementation-specific | Implementation-specific |
| Reboot | `TYPE_REQ_REBOOT` | `req_resp_empty` or `req_bl_cmd` | `TYPE_RESP_REBOOT` | `req_resp_empty` |
| Read neighbors | `TYPE_REQ_NEIGHBORS` | `req_resp_empty` | `TYPE_RESP_NEIGHBORS` | `resp_neighbor_info` |
| Write DT4I URL | `TYPE_REQ_WRITE_DT4I` | `req_resp_dt4i` | Implementation-specific | Implementation-specific |
| Read DT4I URL | `TYPE_REQ_READ_DT4I` | `req_resp_empty` | `TYPE_RESP_READ_DT4I` | `req_resp_dt4i` |
| Write display-connection state | `TYPE_REQ_WRITE_DISP_CONN_STATUS` | `req_disp_conn_status` | `TYPE_RESP_WRITE_DISP_CONN_STATUS` | `req_resp_empty` |
| Read firmware information | `TYPE_REQ_READ_FW_UPDATE_INFO` | `req_resp_empty` | `TYPE_RESP_READ_FW_UPDATE_INFO` | `resp_fw_update_info` |
| Execute memory command | `TYPE_REQ_MEMORY_CMD` | `req_mem_cmd` | `TYPE_RESP_MEMORY_CMD` when applicable | `resp_mem_cmd` |
| Identify device | `TYPE_REQ_IDENTIFY` | `req_identify` | Implementation-specific | Implementation-specific |
| Start discovery | `TYPE_REQ_START_DISCOVERY` | `req_resp_empty` | Implementation-specific | Implementation-specific |
| Control LEDs | `TYPE_REQ_LED_CTRL` | `req_led_ctrl` | Implementation-specific | Implementation-specific |

A response type is only listed when a corresponding response `MsgType` is defined in `sys.proto`.

## `/sys/cfg`: System configuration

### Purpose

The system configuration controls device-level settings such as:

- Controller-connection failure timeout
- Device name
- RS-485 termination
- RS-485 biasing
- Password or PIN slots
- Display language
- Local setup permission

### `SysCfgReq`

Used with:

```text
PATCH /sys/cfg
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `ctrl_connect_failure_timeout_s` | 1 | `Uint32Setting` | Configures the controller-connection failure timeout in seconds |
| `name` | 2 | `String64Setting` | Configures the device name with a maximum length of 64 characters |
| `rs485_ch1_termination` | 3 | `BooleanSetting` | Enables or disables termination on RS-485 channel 1 |
| `rs485_ch1_biasing` | 4 | `BooleanSetting` | Enables or disables bias resistors on RS-485 channel 1 |
| `rs485_ch2_termination` | 5 | `BooleanSetting` | Enables or disables termination on RS-485 channel 2 |
| `rs485_ch2_biasing` | 6 | `BooleanSetting` | Enables or disables bias resistors on RS-485 channel 2 |
| `password1` | 7 | `Uint64Setting` | Configures password or PIN storage slot 1 |
| `password2` | 8 | `Uint64Setting` | Configures password or PIN storage slot 2 |
| `password3` | 9 | `Uint64Setting` | Configures password or PIN storage slot 3 |
| `language` | 11 | `Language` | Selects the language used by the connected display |
| `local_setup_enabled` | 12 | `BooleanSetting` | Enables or disables local device configuration through a connected display |

Field number 10 is reserved and must not be reused.

`password1`, `password2`, and `password3` are opaque `uint64` values. Their functional assignment and encoding are not defined by `sys.proto`.

### `SysCfgResp`

Returned by:

```text
GET /sys/cfg
PATCH /sys/cfg
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `ctrl_connect_failure_timeout_s` | 1 | `uint32` | Effective controller-connection failure timeout in seconds |
| `name` | 2 | `string` | Effective device name with a maximum length of 64 characters |
| `rs485_ch1_termination` | 3 | `bool` | Effective termination state of RS-485 channel 1 |
| `rs485_ch1_biasing` | 4 | `bool` | Effective biasing state of RS-485 channel 1 |
| `rs485_ch2_termination` | 5 | `bool` | Effective termination state of RS-485 channel 2 |
| `rs485_ch2_biasing` | 6 | `bool` | Effective biasing state of RS-485 channel 2 |
| `password1` | 7 | `uint64` | Stored value of password or PIN slot 1 |
| `password2` | 8 | `uint64` | Stored value of password or PIN slot 2 |
| `password3` | 9 | `uint64` | Stored value of password or PIN slot 3 |
| `linkbus_address` | 10 | `uint64` | Current internal Linkbus address |
| `language` | 11 | `Language` | Effective display language |
| `local_setup_enabled` | 12 | `bool` | Effective local-setup state |

The API does not define `password1`, `password2`, or `password3` as hashes.

Clients must treat these fields as security-sensitive opaque values. They should not be logged, displayed, or transmitted outside the intended trusted environment.

`linkbus_address` is read-only through this endpoint. `SysCfgReq` does not define a writable field with field number 10.

### `Language`

| Name | Value | Description |
|---|---:|---|
| `DONT_CHANGE` | 0 | Retains the currently configured language |
| `EN_US` | 1 | English, United States |
| `DE_DE` | 2 | German, Germany |
| `FR_FR` | 3 | French, France |
| `IT_IT` | 4 | Italian, Italy |

### Controller-connection failure timeout

`ctrl_connect_failure_timeout_s` defines the monitoring period for communication with the control unit.

When communication remains interrupted for the configured period, outputs configured to use a default value after communication loss may switch to their configured default value.

The timeout is configured at system level. Each output independently determines whether it uses a default value through:

```text
use_default_val_on_failure
```

### RS-485 termination and biasing

For each supported RS-485 channel:

- Termination activates or deactivates the bus terminator.
- Biasing activates or deactivates the bias resistors.

### Local setup

`local_setup_enabled` controls whether device properties may be configured through a connected display.

| Value | Meaning |
|:---:|---|
| `false` | Device properties cannot be configured through the connected display |
| `true` | Device properties may be configured through the connected display |

This system-level setting is separate from per-channel settings with similar names.

## `/sys/discover`: Device discovery

### Purpose

The discovery endpoint controls device discovery and, depending on the selected option, address assignment for discovered devices.

Used with:

```text
PATCH /sys/discover
```

### `ReqDiscover`

| Field | Number | Type | Description |
|---|---:|---|---|
| `option` | 1 | `DiscoverOption` | Selects the requested discovery operation |

### `DiscoverOption`

| Name | Value | Description |
|---|---:|---|
| `OPTION_GET_STATUS` | 0 | Queries the current discovery status |
| `OPTION_START_READ_ONLY` | 1 | Starts discovery without requesting address assignment |
| `OPTION_START_ADDRESS_ALL` | 2 | Starts discovery with address assignment for all applicable devices |
| `OPTION_GET_RESULT` | 3 | Retrieves the discovery result |
| `OPTION_ABORT` | 4 | Aborts the active discovery operation |

### `RespDiscover`

| Field | Number | Type | Description |
|---|---:|---|---|
| `status` | 1 | `StatusCode` | Current discovery state |
| `progress` | 2 | `uint32` | Discovery progress value |
| `discover_entries` | 3 | Repeated `DiscoverEntry` | Discovered-device entries, maximum 32 |
| `error` | 4 | `ErrorCode` | Host-side discovery error |

The unit and range of `progress` are not defined by `sys.proto`.

### `StatusCode`

| Name | Value | Description |
|---|---:|---|
| `STATUS_IDLE` | 0 | Discovery is idle, completed, or aborted |
| `STATUS_BUSY` | 1 | Discovery is active |

### `DiscoverEntry`

| Field | Number | Type | Length | Description |
|---|---:|---|---:|---|
| `ip_address` | 1 | `bytes` | 4 bytes | IPv4 address of the discovered device |
| `error` | 4 | `ErrorCodeEntry` | Not applicable | Entry-specific discovery error |

Field numbers 2 and 3 are reserved and must not be reused.

### `ErrorCodeEntry`

| Name | Value | Description |
|---|---:|---|
| `ERR_OK` | 0 | No entry-specific error |
| `ERR_INFO` | 1 | Failed to retrieve entry information |
| `ERR_TABLE_LEN` | 2 | Failed to retrieve the entry table length |
| `ERR_TABLE_GET` | 3 | Failed to retrieve the entry table |

### Discovery `ErrorCode`

| Name | Value | Description |
|---|---:|---|
| `ERR_HOST_OK` | 0 | No host-side error |
| `ERR_HOST_INFO` | 1 | Failed to retrieve host information |
| `ERR_HOST_TABLE_LEN` | 2 | Failed to retrieve the host table length |
| `ERR_HOST_TABLE_GET` | 3 | Failed to retrieve the host table |
| `ERR_HOST_GET_IP_SETTINGS` | 4 | Failed to read IP settings |
| `ERR_HOST_SET_IP_SETTINGS` | 5 | Failed to write IP settings |

## `/sys/audio`: Audio control

### Purpose

The audio endpoint controls system, identification, button, and alarm sounds.

Used with:

```text
PATCH /sys/audio
```

### `ReqAudio`

| Field | Number | Type | Description |
|---|---:|---|---|
| `action` | 1 | `Action` | Selects the requested audio operation |
| `sequence` | 2 | `bytes` | Encoded audio sequence with a fixed length of 131 bytes |

`sequence` is used when `action` is `PLAY_SEQUENCE`.

The binary encoding of `sequence` is not defined by its Protobuf field type.

### Audio `Action`

| Name | Value | Description |
|---|---:|---|
| `FAILURE` | 0 | Plays the failure sound |
| `SUCCESS` | 1 | Plays the success sound |
| `IDENTIFY` | 2 | Plays the identification sound |
| `ALARM_1` | 3 | Plays alarm sound 1 |
| `ALARM_2` | 4 | Plays alarm sound 2 |
| `ALARM_3` | 5 | Plays alarm sound 3 |
| `BUTTON_PRESSED` | 6 | Plays the button-pressed sound |
| `BUTTON_RELEASED` | 7 | Plays the button-released sound |
| `PLAY_SEQUENCE` | 8 | Plays the encoded audio sequence |
| `MUTE` | 9 | Mutes audio output |
| `UNMUTE` | 10 | Unmutes audio output |

### `RespAudio`

| Field | Number | Type | Description |
|---|---:|---|---|
| `status` | 1 | `Status` | Current audio state |
| `error` | 2 | `ErrorCode` | Audio-operation error |

### Audio `Status`

| Name | Value | Description |
|---|---:|---|
| `STATUS_IDLE` | 0 | Audio system is idle |
| `STATUS_BUSY` | 1 | Audio system is busy |

### Audio `ErrorCode`

| Name | Value | Description |
|---|---:|---|
| `ERR_OK` | 0 | No audio error |
| `ERR_PLAYING` | 1 | Audio playback is already active |
| `ERR_LOW_PRIO` | 2 | Request rejected because of lower priority |
| `ERR_MUTE` | 3 | Audio output is muted or the mute operation failed |
| `ERR_UNMUTE` | 4 | The unmute operation failed |

## `/sys/signal`: System signal states

### `RespSignal`

Returned by:

```text
GET /sys/signal
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `identify` | 1 | `bool` | Indicates whether device identification is active |
| `update` | 2 | `bool` | Indicates the firmware-update signal state |
| `knx_prog` | 3 | `bool` | Indicates the KNX programming signal state |

## `/sys/load`: System load information

### `RespLoad`

Returned by:

```text
GET /sys/load
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `daemon_load` | 1 | `uint32` | Load value of the system daemon |
| `station_load` | 2 | `uint32` | Load value of the station |

`sys.proto` does not define:

- The unit of the values
- Their scaling
- Their averaging period
- Their sampling interval

Clients must not interpret these values as percentages unless another specification defines the encoding.

## `/sys/ip`: TCP/IP configuration

### `AdapterId`

| Name | Value | Description |
|---|---:|---|
| `EN0` | 0 | Ethernet adapter `en0` |
| `EN1` | 1 | Ethernet adapter `en1` |

### `Ipv4`

| Field | Number | Type | Length | Description |
|---|---:|---|---:|---|
| `ip_address` | 1 | `bytes` | 4 bytes | IPv4 address |
| `subnet_mask_cidr` | 2 | `uint32` | Not applicable | IPv4 prefix length in CIDR format, valid range 0 through 32 |
| `default_gateway` | 3 | `bytes` | 4 bytes | IPv4 default gateway |
| `is_dhcp` | 4 | `bool` | Not applicable | `true` for DHCP assignment and `false` for static addressing |

### `Ipv6`

| Field | Number | Type | Length | Description |
|---|---:|---|---:|---|
| `ip_address` | 1 | `bytes` | 16 bytes | IPv6 address |
| `prefix_length_cidr` | 2 | `uint32` | Not applicable | IPv6 prefix length in CIDR format, valid range 0 through 128 |
| `default_gateway` | 3 | `bytes` | 16 bytes | IPv6 default gateway |
| `is_dhcp` | 4 | `bool` | Not applicable | `true` for DHCP assignment and `false` for static addressing |

### `Adapter`

| Field | Number | Type | Description |
|---|---:|---|---|
| `id` | 1 | `AdapterId` | Adapter identifier |
| `enabled` | 2 | `bool` | Indicates whether the adapter is enabled |
| `can_disable` | 3 | `bool` | Indicates whether `enabled` can be written |
| `is_readonly` | 4 | `bool` | Indicates whether `ipv4`, `enabled`, and `ipv6_enabled` are read-only |
| `ipv6_enabled` | 5 | `bool` | `true` when the IPv6 configuration applies and `false` when the IPv4 configuration applies |
| `can_disable_ipv6` | 6 | `bool` | Indicates whether `ipv6_enabled` can be written |
| `ipv4` | 7 | `Ipv4` | Current IPv4 configuration |
| `ipv6` | 8 | `Ipv6` | Current IPv6 configuration |

Clients must check the capability fields before presenting an adapter setting as writable:

- `can_disable`
- `is_readonly`
- `can_disable_ipv6`

### `SetAdapter`

| Field | Number | Type | Description |
|---|---:|---|---|
| `id` | 1 | `AdapterId` | Identifies the adapter to modify |
| `enable` | 2 | `BooleanSetting` | Changes `Adapter.enabled` |
| `ipv6_enable` | 3 | `BooleanSetting` | Changes `Adapter.ipv6_enabled` |
| `ip_address` | 5 | `Ipv4Setting` | Changes `Adapter.ipv4.ip_address` |
| `subnet_mask_cidr` | 6 | `Uint32Setting` | Changes `Adapter.ipv4.subnet_mask_cidr` |
| `default_gateway` | 7 | `Ipv4Setting` | Changes `Adapter.ipv4.default_gateway`; setting `0.0.0.0` removes the default gateway |
| `set_dhcp` | 8 | `BooleanSetting` | Changes `Adapter.ipv4.is_dhcp` |

Field number 4 is not used by `SetAdapter`.

IPv6 address, prefix, gateway, and DHCP fields are returned by `Adapter` but cannot be changed through the currently defined `SetAdapter` fields.

### `RespTcpIpSettings`

Returned by:

```text
GET /sys/ip
PATCH /sys/ip
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `hostname` | 1 | `string` | System hostname with a maximum length of 64 characters |
| `adapters` | 2 | Repeated `Adapter` | Network adapters with a maximum of two entries |
| `eee` | 3 | `bool` | Indicates whether the adapters operate in eco mode |

`hostname` and `eee` are returned by `RespTcpIpSettings` but cannot be changed through `ReqTcpIpSettings`.

### `ReqTcpIpSettings`

Used with:

```text
PATCH /sys/ip
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `adapters` | 1 | Repeated `SetAdapter` | Adapter changes with a maximum of two entries |

Only fields represented by `SetAdapter` are writable through this endpoint.

## `/sys/time`: System-local time

### `RespTime`

Returned by:

```text
GET /sys/time
PATCH /sys/time
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `local_date_time` | 1 | `uint64` | Current local date and time in Unix timestamp format |

### `ReqTime`

Used with:

```text
PATCH /sys/time
```

| Field | Number | Type | Description |
|---|---:|---|---|
| `local_date_time` | 1 | `Uint64Setting` | Configures the local date and time as a Unix timestamp |

Changing the system time may affect:

- Log timestamps
- Certificate validation
- Scheduled operations
- Time-dependent communication
- Time-series data

The field is described as a local Unix timestamp. Time-zone and daylight-saving handling are determined by the system implementation.

## Device information

Device information is read through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_READ_INFO
payload.req_resp_empty = {}
```

### Response

```text
msg_type = TYPE_RESP_READ_INFO
payload.req_resp_info = {...}
```

### `ReqRespInfo`

| Field | Number | Type | Length or limit | Description |
|---|---:|---|---|---|
| `model_name` | 1 | `string` | Maximum 64 characters | Device model designation |
| `model_number` | 2 | `string` | Maximum 16 characters | Device order or model number |
| `serial_no` | 5 | `string` | Maximum 18 characters | Device serial number |
| `mac_id` | 6 | `bytes` | 6 bytes | Base MAC address or MAC identifier |
| `mac_id_count` | 7 | `uint32` | Not applicable | Number of MAC identifiers associated with the device |
| `host_id` | 8 | `string` | Maximum 32 characters | Host identifier |
| `model_revision` | 13 | `string` | Maximum 2 characters | Model revision |
| `hardware_revision` | 14 | `uint32` | Not applicable | Hardware revision |
| `uuid_plc_next` | 15 | `bytes` | 16 bytes | PLCnext UUID |
| `g_tin` | 16 | `bytes` | 6 bytes | Encoded Global Trade Item Number |
| `knx_serial_no` | 17 | `bytes` | 6 bytes | KNX serial number |
| `dali_hardware_version` | 18 | `bytes` | 2 bytes | DALI hardware version |
| `bldt` | 19 | `string` | Maximum 14 characters | BLDT identifier |

Reserved field numbers:

```text
3
4
9
10
11
12
```

These field numbers must not be reused.

## System status

System status is read through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_READ_STATUS
payload.req_resp_empty = {}
```

### Response

```text
msg_type = TYPE_RESP_READ_STATUS
payload.resp_status = {...}
```

### `RespStatus`

| Field | Number | Type | Description |
|---|---:|---|---|
| `points_in_failure` | 1 | `uint32` | Number of I/O channels currently in the failure state |
| `points_overridden` | 2 | `uint32` | Number of I/O channels currently in the overridden state |
| `display_connected` | 3 | `bool` | Indicates whether a display is connected |
| `ctrl_connected` | 4 | `bool` | Indicates whether the control unit is connected |
| `button_pressed` | 5 | `bool` | Indicates the current device-button state |
| `ctrl_connect_failure` | 6 | `bool` | Indicates a controller-connection failure |

## UI calibration data

UI calibration data is read or written through `PATCH /sys`.

### Read request

```text
msg_type = TYPE_REQ_READ_UI_CAL
payload.req_resp_empty = {}
```

### Read response

```text
msg_type = TYPE_RESP_READ_UI_CAL
payload.req_resp_ui_cal = {...}
```

### Write request

```text
msg_type = TYPE_REQ_WRITE_UI_CAL
payload.req_resp_ui_cal = {...}
```

### `ReqRespUiCal`

| Field | Number | Type | Description |
|---|---:|---|---|
| `res_9k6_ai` | 1 | `double` | Calibration coefficient for the 9.6 kΩ AI path |
| `res_9k6_bi` | 2 | `double` | Calibration coefficient for the 9.6 kΩ BI path |
| `res_9k6_au` | 3 | `double` | Calibration coefficient for the 9.6 kΩ AU path |
| `res_9k6_bu` | 4 | `double` | Calibration coefficient for the 9.6 kΩ BU path |
| `res_180k_ai` | 5 | `double` | Calibration coefficient for the 180 kΩ AI path |
| `res_180k_bi` | 6 | `double` | Calibration coefficient for the 180 kΩ BI path |
| `res_180k_au` | 7 | `double` | Calibration coefficient for the 180 kΩ AU path |
| `res_180k_bu` | 8 | `double` | Calibration coefficient for the 180 kΩ BU path |

## Physical neighbor information

Physical neighbor information is read through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_NEIGHBORS
payload.req_resp_empty = {}
```

### Response

```text
msg_type = TYPE_RESP_NEIGHBORS
payload.resp_neighbor_info = {...}
```

### `RespNeighborInfo`

| Field | Number | Type | Description |
|---|---:|---|---|
| `local_mac` | 1 | `bytes` | MAC address of the local device with a fixed length of 6 bytes |
| `neighbor_entries` | 2 | Repeated `NeighborEntry` | Physical neighbor entries with a maximum of 32 entries |

### `NeighborEntry`

| Field | Number | Type | Description |
|---|---:|---|---|
| `local_phy` | 1 | `uint32` | Local physical interface identifier |
| `remote_phy` | 2 | `uint32` | Physical interface identifier reported for the remote side |
| `remote_mac` | 3 | `bytes` | MAC address of the neighboring device with a fixed length of 6 bytes |

The message describes physical neighbor relationships.

It does not provide:

- Higher-level device names
- IP addresses
- Routing information

## DT4I configuration

The DT4I URL is read or written through `PATCH /sys`.

### Read request

```text
msg_type = TYPE_REQ_READ_DT4I
payload.req_resp_empty = {}
```

### Read response

```text
msg_type = TYPE_RESP_READ_DT4I
payload.req_resp_dt4i = {
  "dt4iUrl": "..."
}
```

### Write request

```text
msg_type = TYPE_REQ_WRITE_DT4I
payload.req_resp_dt4i = {
  "dt4iUrl": "..."
}
```

### `ReqRespDt4i`

| Field | Number | Type | Description |
|---|---:|---|---|
| `dt4i_url` | 1 | `string` | DT4I URL with a maximum length of 256 characters |

## Display-connection state

The reported display-connection state is changed through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_WRITE_DISP_CONN_STATUS
payload.req_disp_conn_status = {
  "displayConnected": true
}
```

### Response

```text
msg_type = TYPE_RESP_WRITE_DISP_CONN_STATUS
payload.req_resp_empty = {}
```

### `ReqDispConnStatus`

| Field | Number | Type | Description |
|---|---:|---|---|
| `display_connected` | 1 | `bool` | Requested display-connection state |

This operation updates the reported display-connection state.

It is separate from `local_setup_enabled`, which controls whether configuration through a connected display is permitted.

## Firmware update information

Firmware information is read through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_READ_FW_UPDATE_INFO
payload.req_resp_empty = {}
```

### Response

```text
msg_type = TYPE_RESP_READ_FW_UPDATE_INFO
payload.resp_fw_update_info = {...}
```

### `FwImageInfo`

| Field | Number | Type | Description |
|---|---:|---|---|
| `bldt` | 1 | `string` | Firmware image BLDT identifier with a maximum length of 14 characters |
| `major` | 2 | `uint32` | Major version component |
| `minor` | 3 | `uint32` | Minor version component |
| `revision` | 4 | `uint32` | Revision version component |
| `build` | 5 | `uint32` | Build version component |
| `size` | 6 | `uint32` | Firmware image size |

### `RespFwUpdateInfo`

| Field | Number | Type | Description |
|---|---:|---|---|
| `bldt` | 1 | `string` | Device BLDT identifier with a maximum length of 14 characters |
| `min_major` | 2 | `uint32` | Minimum supported major firmware version |
| `min_minor` | 3 | `uint32` | Minimum supported minor firmware version |
| `fw_primary` | 4 | `FwImageInfo` | Primary firmware image information |
| `fw_update` | 5 | `FwImageInfo` | Update firmware image information |
| `fw_failsafe` | 6 | `FwImageInfo` | Failsafe firmware image information |
| `fw_display` | 7 | `FwImageInfo` | Display firmware image information |
| `last_bl_action` | 8 | `BlAction` | Last reported bootloader action |
| `bl_major` | 9 | `uint32` | Bootloader major version |
| `bl_minor` | 10 | `uint32` | Bootloader minor version |
| `ext_uif_hw_version` | 11 | `uint32` | External UIF hardware version |
| `reset_cause` | 12 | `uint32` | Reported reset-cause value |

### `BlAction`

| Name | Value | Description |
|---|---:|---|
| `BL_ACTION_NO` | 0 | No bootloader action |
| `BL_ACTION_COPY_UPDATE` | 1 | Copy-update action |
| `BL_ACTION_AUTO_REPAIR` | 2 | Automatic repair action |
| `BL_ACTION_MANUAL_REPAIR` | 3 | Manual repair action |
| `BL_ACTION_CMD_COPY_FAILSAFE` | 4 | Commanded copy of the failsafe image |
| `BL_ACTION_CMD_SAVE_FAILSAFE` | 5 | Commanded save of the failsafe image |

## Reboot and bootloader commands

A reboot is requested through `PATCH /sys`.

### Normal reboot request

```text
msg_type = TYPE_REQ_REBOOT
payload.req_resp_empty = {}
```

### Reboot with bootloader command

```text
msg_type = TYPE_REQ_REBOOT
payload.req_bl_cmd = {
  "blCmd": "BL_CMD_COPY_FAILSAFE"
}
```

### Response

```text
msg_type = TYPE_RESP_REBOOT
payload.req_resp_empty = {}
```

### `RequBlCmd`

The message name `RequBlCmd` is defined by `sys.proto`.

| Field | Number | Type | Description |
|---|---:|---|---|
| `bl_cmd` | 1 | `BlCmd` | Requested bootloader command |

### `BlCmd`

| Name | Value | Description |
|---|---:|---|
| `BL_CMD_NO` | 0 | No bootloader command |
| `BL_CMD_COPY_FAILSAFE` | 1 | Requests copying of the failsafe image |
| `BL_CMD_SAVE_FAILSAFE` | 2 | Requests saving of the failsafe image |

The result of a bootloader command is reported by:

```text
RespFwUpdateInfo.last_bl_action
```

These commands affect bootloader-managed firmware storage and must not be treated as ordinary application settings.

## Memory commands

Memory commands are executed through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_MEMORY_CMD
payload.req_mem_cmd = {
  "memCmd": "MEM_CMD_GET_FIX_PROTECT_LEVEL"
}
```

### Response

Commands that return the fixed-memory protection level use:

```text
msg_type = TYPE_RESP_MEMORY_CMD
payload.resp_mem_cmd = {
  "fixProtectLevel": "FIX_UNPROTECT"
}
```

### `ReqMemoryCmd`

| Field | Number | Type | Description |
|---|---:|---|---|
| `mem_cmd` | 1 | `MemCmd` | Requested memory operation |

### `MemCmd`

| Name | Value | Description |
|---|---:|---|
| `MEM_CMD_NO` | 0 | No memory command |
| `MEM_CMD_CLEAR_CFG` | 1 | Clears configuration memory |
| `MEM_CMD_CLEAR_RUN` | 2 | Clears runtime memory |
| `MEM_CMD_CLEAR_ALL` | 3 | Clears configuration and runtime memory |
| `MEM_CMD_SET_FIX_UNPROTECT` | 4 | Sets fixed-memory protection to unprotected |
| `MEM_CMD_SET_FIX_PROTECT_REVERSIBLE` | 5 | Sets reversible fixed-memory protection |
| `MEM_CMD_SET_FIX_PROTECT_PERMANENT` | 6 | Sets permanent fixed-memory protection |
| `MEM_CMD_GET_FIX_PROTECT_LEVEL` | 7 | Reads the fixed-memory protection level |

### `RespMemoryCmd`

| Field | Number | Type | Description |
|---|---:|---|---|
| `fix_protect_level` | 1 | `FixProtectLevel` | Current fixed-memory protection level |

### `FixProtectLevel`

| Name | Value | Description |
|---|---:|---|
| `FIX_UNPROTECT` | 0 | Fixed memory is unprotected |
| `FIX_PROTECT_REVERSIBLE` | 1 | Fixed memory has reversible protection |
| `FIX_PROTECT_PERMANENT` | 2 | Fixed memory has permanent protection |

### ⚠️ Destructive memory commands

The following memory commands may cause data loss or irreversible configuration changes:

| Command | Impact |
|---|---|
| `MEM_CMD_CLEAR_CFG` | Clears the configuration memory |
| `MEM_CMD_CLEAR_RUN` | Clears the runtime memory |
| `MEM_CMD_CLEAR_ALL` | Clears both configuration and runtime memory |
| `MEM_CMD_SET_FIX_PROTECT_PERMANENT` | Sets permanent fixed-memory protection; cannot be reversed |

The Protobuf schema does not define authorization, confirmation, or recovery procedures for these operations. Clients must implement appropriate safeguards before sending these commands.

## Device identification

Device identification is controlled through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_IDENTIFY
payload.req_identify = {
  "durationS": 30
}
```

### `ReqIdentify`

| Field | Number | Type | Description |
|---|---:|---|---|
| `duration_s` | 1 | `uint32` | Requested identification duration in seconds; `0` switches identification off |

## Discovery start message

In addition to the dedicated `/sys/discover` endpoint, `SysMessage` defines a generic start-discovery request.

### Request

```text
msg_type = TYPE_REQ_START_DISCOVERY
payload.req_resp_empty = {}
```

`sys.proto` does not define a corresponding generic discovery response `MsgType`.

Detailed discovery control and results are provided by:

```text
PATCH /sys/discover
```

## LED control

LED control is performed through `PATCH /sys`.

### Request

```text
msg_type = TYPE_REQ_LED_CTRL
payload.req_led_ctrl = {
  "ledCmd": "LED_CMD_STATUS",
  "daemonLoad": 0,
  "stationLoad": 0
}
```

### `ReqLedCtrl`

| Field | Number | Type | Description |
|---|---:|---|---|
| `led_cmd` | 1 | `LedCmd` | Requested LED operation |
| `daemon_load` | 2 | `uint32` | Daemon load value used by the LED status operation |
| `station_load` | 3 | `uint32` | Station load value used by the LED status operation |

### `LedCmd`

| Name | Value | Description |
|---|---:|---|
| `LED_CMD_NO` | 0 | No LED command |
| `LED_CMD_UPDATE_ON` | 1 | Switches the update LED on |
| `LED_CMD_UPDATE_OFF` | 2 | Switches the update LED off |
| `LED_CMD_STATUS` | 3 | Updates the LED status using the supplied load values |
| `LED_CMD_PROG_ON` | 4 | Switches the programming LED on |
| `LED_CMD_PROG_OFF` | 5 | Switches the programming LED off |

The exact conversion of `daemon_load` and `station_load` into an LED indication is implementation-specific.

## Login messages

`sys.proto` defines login request and response message types, but the current `SysMessage.MsgType` enum and `SysMessage.payload` oneof do not expose fields for these messages.

They therefore cannot be transported through the currently defined `SysMessage` container without a schema extension or another transport definition.

### `ReqLogin`

| Field | Number | Type | Description |
|---|---:|---|---|
| `user_name` | 1 | `string` | User name with a maximum length of 16 characters |
| `pw_hash` | 2 | `string` | Password hash with a maximum length of 64 characters |

### `RespLogin`

| Field | Number | Type | Description |
|---|---:|---|---|
| `code` | 1 | `ErrorCode` | Login result |
| `user_group` | 2 | `UserGroup` | Assigned user group |
| `rights` | 3 | `string` | Encoded rights description with a maximum length of 64 characters |

### Login `ErrorCode`

| Name | Value | Description |
|---|---:|---|
| `ERR_OK` | 0 | Login successful |
| `ERR_NOK` | 1 | Login failed |

### `UserGroup`

| Name | Value | Description |
|---|---:|---|
| `USER_GROUP_UNAUTHENTICATED` | 0 | Unauthenticated display access |
| `USER_GROUP_USER` | 1 | Standard display user |
| `USER_GROUP_SEVICE` | 2 | Service personnel using the display |
| `USER_GROUP_CTRL` | 3 | Controller |
| `USER_GROUP_ADMIN` | 4 | Display administrator |

`USER_GROUP_SEVICE` is the enum name defined by `sys.proto`.

## Shared enums and message types

For more information, see [`shared/README.md`](../shared/README.md).

- `Setting`
  - Used by the corresponding `*Setting` messages in `SysCfgReq`, `ReqTime`, `SetAdapter`
- `BooleanSetting`
  - Used in `SysCfgReq`, `SetAdapter`
- `Uint32Setting`, `Uint64Setting`, `String64Setting`, `Ipv4Setting`
  - Used in `SysCfgReq`, `SetAdapter`, `ReqTime`

## Implementation-specific behavior

The following details are not fully defined by `sys.proto`:

- Scaling and unit of discovery progress
- Scaling and unit of daemon and station load values
- Binary format of audio sequences
- Encoding and functional assignment of password or PIN slots
- Time-zone and daylight-saving handling of the local Unix timestamp
- Conversion of LED status load values into physical LED indications
- Response behavior for operations without a corresponding response `MsgType`
- Authorization and recovery procedures for destructive memory commands

Clients must not infer these details solely from the Protobuf scalar types.

