
# 📘 CatanIO – API Documentation for relay outputs (DOR)

## Overview

This API defines the configuration and control of **relays** via **CoAP**.  
Communication is handled using **Protocol Buffers** messages.

## Dependencies
- `shared.proto` → Enums, settings, helper structures
- `uoi.proto` → UOI-specific messages and enums

---

## Endpoints
| Method | Example URL                              | Message |
|---------|-------------------------------------------|-----------|
| `GET`   | `coap://<ip>:<port>/dor/<channel>/cfg`    |  `DorCfgResp` |
| `PATCH` | `coap://<ip>:<port>/dor/<channel>/cfg`    |  `DorCfgReq` |
| `GET`   | `coap://<ip>:<port>/dor/<channel>`        |  `DorResp` |
| `PATCH` | `coap://<ip>:<port>/dor/<channel>`        |  `DorReq` |
---

## ✅ Readable and Writable Values

| Category | Field name | Data type | Readable (GET) | Writable (PATCH) |
|----------|------------|-----------|----------------|------------------|
| General | enabled | bool / BooleanSetting | ✅ | ✅ |
|        | name | string (max 64) / String64Setting | ✅ | ✅ |
|        | override_enabled | bool / BooleanSetting | ✅ | ✅ |
|        | override_cancelable | bool / BooleanSetting | ✅ | ✅ |
| Signal | inverted | bool / BooleanSetting | ✅ | ✅ |
|        | use_default_val_on_start | bool / BooleanSetting | ✅ | ✅ |
|        | use_default_val_on_failure | bool / BooleanSetting | ✅ | ✅ |
|        | digital_default_val | bool / BooleanSetting | ✅ | ✅ |
| Texts  | false_text | string (max 16) / String16Setting | ✅ | ✅ |
|        | true_text | string (max 16) / String16Setting | ✅ | ✅ |
| Status | configured | bool / BooleanSetting | ✅ | ✅ |
|        | local_setup_enabled | bool / BooleanSetting | ✅ | ✅ |
|        | test_status | TestStatus | ✅ | ✅ |
| Override / Values | digital_set_val | bool / BooleanSetting | ✅ | ✅ |
|        | override_action | OverrideAction | ❌ | ✅ |
|        | override_duration_ms | uint32 | ❌ | ✅ |
|        | digital_override_val | BooleanSetting | ❌ | ✅ |
|        | counter_init_val | Uint64Setting | ❌ | ✅ |
|        | init_on_time_s | Uint32Setting | ❌ | ✅ |
| Measurements | digital_val | bool | ✅ | ❌ |
|        | counter_val | uint64 | ✅ | ❌ |
|        | on_time_s | uint32 | ✅ | ❌ |

---

## DOR enum

### OverrideAction
Used in: `DorReq.override_action`

| Value | Name | Description |
|------|------|-------------|
| 0 | OVERRIDE_ACTION_NONE | No action |
| 1 | OVERRIDE_ACTION_START | Start override |
| 2 | OVERRIDE_ACTION_STOP | Stop override |
| 3 | OVERRIDE_ACTION_CANCEL | Cancel override |

## Shared enums
For more information see shared.proto

* PointStatus
Used in: `UiResp.status`

* IoErrorCode
Used in: `UiResp.error_code`, `IoErrorCodeSetting.val`

* OverrideAction
Used in: `UiReq.override_action`

* TestStatus
Used in: `UiCfgReq.test_status`, `UiCfgResp.test_status`, `UiResp.test_status`

* Setting
Used in: all `*Setting` messages




