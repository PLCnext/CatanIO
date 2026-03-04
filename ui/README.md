
# 📘 CatanIO – API Documentation universal inputs (UI)

## Overview
This API defines the configuration and control of **Universal Inputs (UI)** via **CoAP**.  
Communication is handled using **Protocol Buffers** messages.

## Dependencies

- `shared.proto` → Enums, settings, helper structures
- `uoi.proto` → UOI-specific messages and enums

---

## Endpoints
| Method | Example URL                            | Massage |
|---------|-------------------------------------------|-----------|
| `GET`   | `coap://<ip>:<port>/ui/<channel>/cfg`    | `UiCfgResp` |
| `PATCH` | `coap://<ip>:<port>/ui/<channel>/cfg`    | `UiCfgReq` |
| `GET`   | `coap://<ip>:<port>/ui/<channel>`        | `UiResp` |
| `PATCH` | `coap://<ip>:<port>/ui/<channel>`        | `UiReq` |

---

## ✅ **Lesbare und Schreibbare Werte**

|Category|Field name|Data type|Readable(GET)|Writable(PATCH)|
|---|---|---|---|---|
|General|enabled|bool/BooleanSetting|✅|✅|
||type|UiType|✅|✅|
||name|string(max64)/String64Setting|✅|✅|
||override_enabled|bool/BooleanSetting|✅|✅|
||override_cancelable|bool/BooleanSetting|✅|✅|
|Signal|inverted|bool/BooleanSetting|✅|✅|
||open_contact|bool/BooleanSetting|✅|✅|
||two_point_calibration|Line/LineSetting|✅|✅|
||characteristic_curve|Curve/CurveSetting|✅|✅|
||unit|string(max16)/String16Setting|✅|✅|
||counter_scale|float/FloatSetting|✅|✅|
|COV|cov_increment|float/FloatSetting|✅|✅|
||min_send_time_ms|uint32/Uint32Setting|✅|✅|
||max_send_time_ms|uint32/Uint32Setting|✅|✅|
|Limits|minimum|float/FloatSetting|✅|✅|
||maximum|float/FloatSetting|✅|✅|
||lower_fault_level|float/FloatSetting|✅|✅|
||upper_fault_level|float/FloatSetting|✅|✅|
|Text|false_text|string(max16)/String16Setting|✅|✅|
||true_text|string(max16)/String16Setting|✅|✅|
|Status|configured|bool/BooleanSetting|✅|✅|
||local_setup_enabled|bool/BooleanSetting|✅|✅|
||test_status|TestStatus|✅|✅|
|Override/Value|counter_init_val|Uint64Setting|❌|✅|
||init_on_time_s|Uint32Setting|❌|✅|
||override_action|OverrideAction|❌|✅|
||override_duration_ms|uint32|❌|✅|
||digital_override_val|BooleanSetting|❌|✅|
||analog_override_val|FloatSetting|❌|✅|
||counter_override_val|Uint64Setting|❌|✅|
|Measurements|digital_val|bool|✅|❌|
||analog_val|float|✅|❌|
||counter_val|uint64|✅|❌|
||on_time_s|uint32|✅|❌|
||sensor_val|float|✅|❌|

---

## UI enum

* UiType
Used in: `UiCfgReq.type`, `UiCfgResp.type`, `UiResp.type`

| Value | Name | Description |
|------|------|-------------|
|0|    UI_TYPE_DONT_CHANGE|inital value |
|1|    UI_TYPE_DEFAULT| default value|
|2|    UI_TYPE_AI_VOLTAGE| Voltage 0..10V|
|3|    UI_TYPE_AI_CURRENT_0_20| Current 0..20mA|
|4|    UI_TYPE_AI_RESISTOR_10K| Resistor 0..10kOhm|
|5|    UI_TYPE_AI_RESISTOR_180K| Resistor 0..180kOhm|
|6|    UI_TYPE_AI_TEMPERATURE_PT1000_DIN| PT1000_DIN|
|7|    UI_TYPE_AI_TEMPERATURE_PT1000_SAMA| PT1000_SAMA|
|8|    UI_TYPE_AI_TEMPERATURE_NI1000_DIN| NI1000_DIN|
|9|    UI_TYPE_AI_TEMPERATURE_NI1000_SAMA| NI1000_SAMA|
|10|   UI_TYPE_AI_TEMPERATURE_NI1000_LG| NI1000_LG|
|11|   UI_TYPE_DI| Digital input |
|12|   UI_TYPE_COUNTER| Counter|
|13|   UI_TYPE_AI_TEMPERATURE_NTC_10K| NTC_10K |
|14|   UI_TYPE_AI_TEMPERATURE_NTC_10K_PRE|NTC_10K_PRE |
|15|   UI_TYPE_AI_TEMPERATURE_NTC_20K|NTC_20K |
|16|   UI_TYPE_AI_CURRENT_4_20| Current 3..20mA|


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

## Example response (GET /ui/\<channel\>)

```json
{
      "type": "UI_TYPE_AI_VOLTAGE",
      "status": "STATUS_FAILURE",
      "errorCode": "ERR_PERIPHERY",
      "overrideRemainingMs": 0,
      "digitalVal": false,
      "analogVal": 0,
      "counterVal": "0",
      "onTimeS": 0,
      "sensorVal": 0,
      "testStatus": "TEST_DONT_CHANGE"
}
```

## Example response (GET /ui/\<channel\>/cfg)

```json
 {
      "enabled": true,
      "type": "UI_TYPE_AI_VOLTAGE",
      "name": "",
      "overrideEnabled": false,
      "overrideCancelable": false,
      "inverted": false,
      "openContact": false,
      "twoPointCalibration": null,
      "characteristicCurve": null,
      "unit": "",
      "counterScale": 1,
      "covIncrement": 0.10000000149011612,
      "minSendTimeMs": 100,
      "maxSendTimeMs": 60000,
      "minimum": 0,
      "maximum": 0,
      "lowerFaultLevel": 0,
      "upperFaultLevel": 0,
      "falseText": "",
      "trueText": "",
      "configured": false,
      "localSetupEnabled": false,
      "testStatus": "TEST_DONT_CHANGE"
}
```
