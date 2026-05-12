
# 📘 CatanIO – API Documentation for DALI

**The module is currently under development; this description is a preliminary draft.**
<!--
## Overview
This API defines the configuration and control of **DALI networks** via **CoAP**.  
Communication is handled using **Protocol Buffers** messages.


## ✅ File Structure
- `shared.proto` → Enums, settings, helper structures
- `dali.proto` → DALI-specific messages and enums

---

## Endpoint
| Method | Example URL                                      | Massage |
|---------|---------------------------------------------------|-----------|
| `GET`   | `coap://<ip>:<port>/dali/<net [1..4]>/cfg`       |  `DaliCfgResp` |
| `PATCH` | `coap://<ip>:<port>/dali/<net [1..4]>/cfg`       |  `DaliCfgReq` |
| `PUT`   | `coap://<ip>:<port>/dali/<net [1..4]>/comm`      |  `DaliTxReq` |
| `GET`   | `coap://<ip>:<port>/dali/<net [1..4]>/comm`      |  `DaliRxRespSeq` |
| `PATCH` | `coap://<ip>:<port>/dali/<net [1..4]>/cmd`       |  `DaliCmd` |
| `GET`   | `coap://<ip>:<port>/dali/<net [1..4]>/gear`      |  `DaliGearResp` |
| `GET`   | `coap://<ip>:<port>/dali/<net [1..4]>/cdev`      |  `DaliCdevResp` |

---

## ✅ Readable and writable values

| Category | Field name | Data type | Readable (GET) | Writable (PATCH) |
|----------|------------|-----------|----------------|------------------|
| General | enabled | bool / BooleanSetting | ✅ | ✅ |
|        | bus_power | bool / BooleanSetting | ✅ | ✅ |
|        | name | string (max 64) / String64Setting | ✅ | ✅ |
|        | override_enabled | bool / BooleanSetting | ✅ | ✅ |
|        | override_cancelable | bool / BooleanSetting | ✅ | ✅ |
| Status | configured | bool / BooleanSetting | ✅ | ✅ |
|        | local_setup_enabled | bool / BooleanSetting | ✅ | ✅ |
|        | test_status | TestStatus | ✅ | ✅ |

---

## ✅ Enums and Usage

### DaliPktType
Used in: `DaliTxReq.type`, `DaliRxRespSeq.pkt_type`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_LEN_16 | Packet length 16 bit |
| 1 | DALI_LEN_24 | Packet length 24 bit |
| 2 | DALI_LEN_25 | Packet length 25 bit |
| 3 | DALI_LEN_20 | Packet length 20 bit |
| 4 | DALI_LEN_32 | Packet length 32 bit |
| 5 | DALI_NO_FF | No forward frame |

---

### DaliPrio
Used in: `DaliTxReq.prio`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_PRIO_2 | Priority 2 |
| 1 | DALI_PRIO_3 | Priority 3 |
| 2 | DALI_PRIO_4 | Priority 4 |
| 3 | DALI_PRIO_5 | Priority 5 |

---

### DaliAnswerType
Used in: `DaliRxResp.DaliRxPkt.answer_type`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_ANSWER_NO | No answer |
| 1 | DALI_ANSWER_VALID | Valid answer |
| 2 | DALI_ANSWER_INVALID | Invalid answer |

---

### DaliPktOrigin
Used in: `DaliRxRespSeq.pkt_origin`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_ORIGIN_EXT | External (bus, sensors) |
| 1 | DALI_ORIGIN_USR | User command |
| 2 | DALI_ORIGIN_COMM | Communication endpoint |

---

### DaliDeviceType
Used in: `DaliCmd.type`, `DaliDeviceCfgResp.device_type`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_FLUORESCENT_LAMP | Fluorescent lamp |
| 6 | DALI_LED_MODULE | LED module |
| 9 | DALI_CONTROL_GEAR | Control gear |
| 13 | DALI_CONTROL_DEVICE | Control device |
| 255 | DALI_MULTIPLE_DEVICE_TYPES | Multiple device types |

---

### DaliProcessStatus
Used in: `DaliGearResp.status`, `DaliCdevResp.status`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_DISCOVERY_NOT_STARTED | Discovery not started |
| 1 | DALI_DISCOVERY_RUNNING | Discovery running |
| 2 | DALI_DISCOVERY_FINISHED | Discovery finished |
| 3 | DALI_DISCOVERY_CANCELED | Discovery canceled |
| 4 | DALI_DISCOVERY_FAILED | Discovery failed |

---

### DiscoveryAddressOption
Used in: `DaliCmd.ReqGear.param.addr_option`

| Value | Name | Description |
|------|------|-------------|
| 0 | DALI_ADDRESS_NONE | No address |
| 1 | DALI_ADDRESS_UNADDRESSED | Unaddressed devices |
| 2 | DALI_ADDRESS_ALL | All devices |



-->