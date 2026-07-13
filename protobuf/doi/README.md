
# 📘 CatanIO – API Documentation for DOI (Digital Output, Digital Input & Counter)

## Overview

This API defines the configuration and control of **digital output and input (DOI)** via **CoAP**.  
Communication is handled using **Protocol Buffers** messages.

## Dependencies
- `shared.proto` → Enums, settings, helper structures
- `uoi.proto` → UOI-specific messages and enums

## Note
- DOI is a subset of UOI and includes **DO**, **DI**, and **Counter**.
- Analog fields such as `analogVal` or `analogOverrideVal` are not included.

---

## Endpoints

| Method | Example URL                                 | Message |
|--------|---------------------------------------------|---------|
| `GET`  | `coap://<ip>:<port>/doi/<channel>/cfg`      | `DoiCfgResp`|
| `PATCH`| `coap://<ip>:<port>/doi/<channel>/cfg`      |`DoiCfgReq`|
| `GET`  | `coap://<ip>:<port>/doi/<channel>`          | `DoiResp`|
| `PATCH`| `coap://<ip>:<port>/doi/<channel>`          |  `DoiReq`|

---

## ✅ Readable and Writable Values

| Category | Field name | Data type | Readable (GET) | Writable (PATCH) |
|----------|------------|-----------|----------------|------------------|
| General | enabled | bool | ✅ | ✅ |
|        | type | UOI_TYPE_DO / UOI_TYPE_DI / UOI_TYPE_COUNTER | ✅ | ✅ |
|        | name | string (max 64) | ✅ | ✅ |
|        | overrideEnabled | bool | ✅ | ✅ |
|        | overrideCancelable | bool | ✅ | ✅ |
| Signal | inverted | bool | ✅ | ✅ |
|        | digitalDefaultVal | bool | ✅ | ✅ |
|        | useDefaultValOnStart | bool | ✅ | ✅ |
|        | useDefaultValOnFailure | bool | ✅ | ✅ |
| Texts  | falseText | string (max 16) | ✅ | ✅ |
|        | trueText | string (max 16) | ✅ | ✅ |
| Status | configured | bool | ✅ | ✅ |
|        | localSetupEnabled | bool | ✅ | ✅ |
|        | testStatus | TestStatus | ✅ | ✅ |
| Override / Values | digitalSetVal | bool | ✅ | ✅ |
|        | overrideAction | OverrideAction | ❌ | ✅ |
|        | overrideDurationMs | uint32 | ❌ | ✅ |
|        | digitalOverrideVal | bool | ❌ | ✅ |
| Measurements | digitalVal | bool | ✅ | ❌ |
|        | counterVal | uint64 | ✅ | ❌ |
|        | onTimeS | uint32 | ✅ | ❌ |

## Enums and Usage

### UOI_TYPE for DOI

| Value | Name | Description |
|------|------|-------------|
| 0 | UOI_TYPE_DONT_CHANGE | No change |
| 1 | UOI_TYPE_DEFAULT | Default |
| 2 | UOI_TYPE_DO | Digital Output |
| 3 | UOI_TYPE_DI | Digital Input |
| 4 | UOI_TYPE_COUNTER | Counter |
| 5 | UOI_TYPE_AO_VOLTAGE | (not used for DOI) |

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

---

## Example response (GET /doi/\<channel\>)
```json
{
    "type": "UOI_TYPE_DO",
    "status": "STATUS_UNCONFIGURED",
    "errorCode": "ERR_OK",
    "overrideRemainingMs": 0,
    "digitalVal": false,
    "counterVal": 0,
    "onTimeS": 0,
    "digitalSetVal": false,
    "testStatus": "TEST_OPEN"
}
```

## Example response (GET /doi/\<channel\>/cfg)
```json
{
    "enabled": false,
    "type": "UOI_TYPE_DO",
    "name": "",
    "overrideEnabled": true,
    "overrideCancelable": false,
    "digitalDefaultVal": false,
    "useDefaultValOnStart": true,
    "useDefaultValOnFailure": true,
    "inverted": false,
    "falseText": "",
    "trueText": "",
    "configured": false,
    "localSetupEnabled": false,
    "testStatus": "TEST_OPEN"
}
```

