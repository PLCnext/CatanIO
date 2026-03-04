
# 📘 CatanIO – API documentation for universal outputs (UOI)

## Overview

This API defines the configuration and control of **universal outputs (UOI)** via **CoAP**.  
Communication is handled using **Protocol Buffers** messages.

## Dependencies

- `shared.proto` → Enums, settings, helper structures
- `uoi.proto` → UOI-specific messages and enums

---

## Endpoint
| Method | Example URL                              | Massage |
|---------|-------------------------------------------|-----------|
| `GET`   | `coap://<ip>:<port>/uoi/<channel>/cfg`    | `UoiCfgResp` |
| `PATCH` | `coap://<ip>:<port>/uoi/<channel>/cfg`    | `UoiCfgReq` |
| `GET`   | `coap://<ip>:<port>/uoi/<channel>`        | `UoiResp` |
| `PATCH` | `coap://<ip>:<port>/uoi/<channel>`        | `UoiReq` |

---

## ✅ Readable and Writable Values

| Category | Field name | Data type | Readable (GET) | Writable (PATCH) |
|----------|------------|-----------|----------------|------------------|
| General | enabled | bool / BooleanSetting | ✅ | ✅ |
|        | type | UoiType | ✅ | ✅ |
|        | name | string (max 64) / String64Setting | ✅ | ✅ |
|        | override_enabled | bool / BooleanSetting | ✅ | ✅ |
|        | override_cancelable | bool / BooleanSetting | ✅ | ✅ |
| Default values | digital_default_val | bool / BooleanSetting | ✅ | ✅ |
|        | analog_default_val | float / FloatSetting | ✅ | ✅ |
|        | use_default_val_on_start | bool / BooleanSetting | ✅ | ✅ |
|        | use_default_val_on_failure | bool / BooleanSetting | ✅ | ✅ |
| Signal | inverted | bool / BooleanSetting | ✅ | ✅ |
|        | two_point_calibration | Line / LineSetting | ✅ | ✅ |
|        | characteristic_curve | Curve / CurveSetting | ✅ | ✅ |
|        | unit | string (max 16) / String16Setting | ✅ | ✅ |
|        | counter_scale | float / FloatSetting | ✅ | ✅ |
| COV | cov_increment | float / FloatSetting | ✅ | ✅ |
|        | min_send_time_ms | uint32 / Uint32Setting | ✅ | ✅ |
|        | max_send_time_ms | uint32 / Uint32Setting | ✅ | ✅ |
| Limits | minimum | float / FloatSetting | ✅ | ✅ |
|        | maximum | float / FloatSetting | ✅ | ✅ |
|        | lower_fault_level | float / FloatSetting | ✅ | ✅ |
|        | upper_fault_level | float / FloatSetting | ✅ | ✅ |
| Texts | false_text | string (max 16) / String16Setting | ✅ | ✅ |
|        | true_text | string (max 16) / String16Setting | ✅ | ✅ |
| Status | configured | bool / BooleanSetting | ✅ | ✅ |
|        | local_setup_enabled | bool / BooleanSetting | ✅ | ✅ |
|        | test_status | TestStatus | ✅ | ✅ |
| Override / Values | digital_set_val | bool / BooleanSetting | ✅ | ✅ |
|        | analog_set_val | float / FloatSetting | ✅ | ✅ |
|        | counter_init_val | Uint64Setting | ❌ | ✅ |
|        | init_on_time_s | Uint32Setting | ❌ | ✅ |
|        | override_action | OverrideAction | ❌ | ✅ |
|        | override_duration_ms | uint32 | ❌ | ✅ |
|        | digital_override_val | BooleanSetting | ❌ | ✅ |
|        | analog_override_val | FloatSetting | ❌ | ✅ |
|        | counter_override_val | Uint64Setting | ❌ | ✅ |
|        | error_code | IoErrorCodeSetting | ❌ | ✅ |
| Measurements | digital_val | bool | ✅ | ❌ |
|        | analog_val | float | ✅ | ❌ |
|        | counter_val | uint64 | ✅ | ❌ |
|        | on_time_s | uint32 | ✅ | ❌ |
|        | actuator_val | float | ✅ | ❌ |

---

## Enums and Usage

### UoiType
Used in: `UoiCfgReq.type`, `UoiCfgResp.type`, `UoiResp.type`

| Value | Name | Description |
|------|------|-------------|
| 0 | UOI_TYPE_DONT_CHANGE | No change |
| 1 | UOI_TYPE_DEFAULT | Default |
| 2 | UOI_TYPE_DO | Digital Output |
| 3 | UOI_TYPE_DI | Digital Input |
| 4 | UOI_TYPE_COUNTER | Counter |
| 5 | UOI_TYPE_AO_VOLTAGE | Analog Output (voltage) |

---
## Shared enums
For more information see shared.proto

* PointStatus \
Used in: `UiResp.status`

* IoErrorCode \
Used in: `UiResp.error_code`, `IoErrorCodeSetting.val`

* OverrideAction \
Used in: `UiReq.override_action`

* TestStatus \
Used in: `UiCfgReq.test_status`, `UiCfgResp.test_status`, `UiResp.test_status`

* Setting \
Used in: all `*Setting` messages

* BooleanSetting, Setting \
Used in: all `*Setting` fields (e.g. digital_set_val, analog_set_val, enabled)


## Example response (GET /uoi/\<channel\>)

```json
{
      "type": "UOI_TYPE_DONT_CHANGE",
      "status": "STATUS_UNCONFIGURED",
      "errorCode": "ERR_OK",
      "overrideRemainingMs": 0,
      "digitalVal": false,
      "analogVal": 0,
      "counterVal": "0",
      "onTimeS": 0,
      "digitalSetVal": false,
      "analogSetVal": 0,
      "actuatorVal": 0,
      "testStatus": "TEST_DONT_CHANGE"
}
```

## Example response (GET /uoi/\<channel\>/cfg)

```json
{
      "enabled": false,
      "type": "UOI_TYPE_DI",
      "name": "",
      "overrideEnabled": true,
      "overrideCancelable": false,
      "digitalDefaultVal": false,
      "analogDefaultVal": 0,
      "useDefaultValOnStart": true,
      "useDefaultValOnFailure": true,
      "inverted": false,
      "twoPointCalibration": null,
      "characteristicCurve": null,
      "unit": "",
      "counterScale": 1,
      "covIncrement": 0,
      "minSendTimeMs": 100,
      "maxSendTimeMs": 0,
      "minimum": null,
      "maximum": null,
      "lowerFaultLevel": null,
      "upperFaultLevel": null,
      "falseText": "",
      "trueText": "",
      "configured": false,
      "localSetupEnabled": false,
      "testStatus": "TEST_OPEN"
}
```
