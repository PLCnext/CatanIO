# 📘 CatanIO – API Documentation shared

## Overview

Shared enums 

# ProtobufSharedTypes(proto3)

## Enums

### PointStatus
|Name|Value|Description|
|---|---|---|
|STATUS_OK|0|valueisvalid|
|STATUS_FAILURE|1|hardwareproblemdetected|
|STATUS_OVERRIDDEN|2|manualoverride|
|STATUS_DISABLED|3|outofservice|
|STATUS_UNCONFIGURED|4|novalidconfiguration|

### IoErrorCode
|Name|Value|
|---|---|
|ERR_OK|0|
|ERR_PERIPHERY|5|
|ERR_RES_HIGH|10|
|ERR_TEMP_HIGH|11|
|ERR_IN_VOLT_HIGH|12|
|ERR_IN_CURR_SHUNT_RES_HIGH|13|
|ERR_OUT_VOLT_HIGH|14|
|ERR_WARN_LIM_HIGH|15|
|ERR_TEMP_LOW|20|
|ERR_NEG_IN_VOLT|21|
|ERR_IN_CURR_SMALL|22|
|ERR_NEG_IN_CURR|23|
|ERR_NEG_IN_CURR_SENSOR_NOT_CONN|24|
|ERR_OUT_VOLT_SMALL|25|
|ERR_WARN_LIM_LOW|26|
|ERR_SENSOR_NOT_CONN|30|
|ERR_LOW_LEVEL_EXCEEDS_VOLT_LIMIT|31|
|ERR_OUT_OVERLOAD|40|
|ERR_VIRT_NULL|50|
|ERR_VIRT_ALARM|51|
|ERR_VIRT_STALE|52|
|ERR_VIRT_DOWN|53|
|ERR_VIRT_FAULT|54|
|ERR_VIRT_DISABLED|55|

### OverrideAction
|Name|Value|
|---|---|
|OVERRIDE_ACTION_NONE|0|
|OVERRIDE_ACTION_START|1|
|OVERRIDE_ACTION_STOP|2|
|OVERRIDE_ACTION_CANCEL|3|

### TestStatus
|Name|Value|
|---|---|
|TEST_DONT_CHANGE|0|
|TEST_OPEN|1|
|TEST_PASSED|2|
|TEST_FAILED|3|
|TEST_BLOCKED|4|
|TEST_SKIPPED|5|

### Setting
|Name|Value|
|---|---|
|SET_DONT_CHANGE|0|
|SET_DEFAULT|1|
|SET_VAL|2|

### BooleanSetting
|Name|Value|
|---|---|
|BOOLEAN_SET_DONT_CHANGE|0|
|BOOLEAN_SET_DEFAULT|1|
|BOOLEAN_SET_FALSE|2|
|BOOLEAN_SET_TRUE|3|

---

## Messages

### SupportPoint
|Field|Type|Tag|
|---|---|---|
|x|float|1|
|y|float|2|

### Line
|Field|Type|Tag|Constraints|
|---|---|---|---|
|vals|repeatedSupportPoint|1|max_count=2|

### Curve
|Field|Type|Tag|Constraints|
|---|---|---|---|
|name|string|2|max_length=64|
|vals|repeatedSupportPoint|3|max_count=20|

### String16Setting
|Field|Type|Tag|Constraints|
|---|---|---|---|
|setting|Setting|1||
|val|string|2|max_length=16|

### String64Setting
|Field|Type|Tag|Constraints|
|---|---|---|---|
|setting|Setting|1||
|val|string|2|max_length=64|

### Uint64Setting
|Field|Type|Tag|
|---|---|---|
|setting|Setting|1|
|val|uint64|2|

### Uint32Setting
|Field|Type|Tag|
|---|---|---|
|setting|Setting|1|
|val|uint32|2|

### FloatSetting
|Field|Type|Tag|
|---|---|---|
|setting|Setting|1|
|val|float|2|

### LineSetting
|Field|Type|Tag|Constraints|
|---|---|---|---|
|setting|Setting|1||
|vals|repeatedSupportPoint|2|max_count=2|

### CurveSetting
|Field|Type|Tag|Constraints|
|---|---|---|---|
|setting|Setting|1||
|name|string|3|max_length=64|
|vals|repeatedSupportPoint|4|max_count=20|

### IoErrorCodeSetting
|Field|Type|Tag|
|---|---|---|
|setting|Setting|1|
|val|IoErrorCode|2|

### Ipv4Setting
|Field|Type|Tag|Constraints|
|---|---|---|---|
|setting|Setting|1||
|val|bytes|2|size=4,fixed|
