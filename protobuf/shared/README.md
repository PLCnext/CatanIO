# CatanIO Shared API Documentation

## Overview

`shared.proto` defines common enums, setting types, diagnostic states, and signal-processing structures for the CatanIO interfaces.

The definitions are shared by the following interfaces:

- UI
- UOI
- DOI
- UV
- DOR
- System
- DALI

The schema uses:

```text
Serialization: Protocol Buffers (proto3)
Constraints:   nanopb options
```

The active `shared.proto` file is authoritative for:

- Enum names and numeric values
- Message names
- Field names
- Field numbers
- Field types
- String-length constraints
- Repeated-field limits
- Fixed byte-array lengths
- Reserved field numbers and enum values

## Dependencies

`shared.proto` imports:

```proto
import "nanopb.proto";
```

The nanopb annotations define constraints required by embedded implementations, including:

- Maximum string lengths
- Maximum repeated-field counts
- Fixed byte-array lengths

## Enums

### `PointStatus`

`PointStatus` represents the general state of an I/O point.

| Name | Value | Description |
|---|---:|---|
| `STATUS_OK` | 0 | The process value is valid |
| `STATUS_FAILURE` | 1 | A hardware problem was detected |
| `STATUS_OVERRIDDEN` | 2 | A manual override is active |
| `STATUS_DISABLED` | 3 | The point is out of service |
| `STATUS_UNCONFIGURED` | 4 | The point does not have a valid configuration |

Clients should evaluate `PointStatus` together with:

```text
error_code
process value
active operating mode
```

The presence of a scalar process value does not guarantee that the value is currently valid.

For example, a response may contain a scalar default value while the point has one of the following states:

```text
STATUS_FAILURE
STATUS_DISABLED
STATUS_UNCONFIGURED
```

### `IoErrorCode`

`IoErrorCode` provides detailed I/O diagnostic information.

| Name | Value | Description |
|---|---:|---|
| `ERR_OK` | 0 | No I/O error |
| `ERR_PERIPHERY` | 5 | Peripheral hardware error |
| `ERR_RES_HIGH` | 10 | Resistance is too high |
| `ERR_TEMP_HIGH` | 11 | Temperature is too high |
| `ERR_IN_VOLT_HIGH` | 12 | Input voltage is too high |
| `ERR_IN_CURR_SHUNT_RES_HIGH` | 13 | Current-input shunt resistance is too high |
| `ERR_OUT_VOLT_HIGH` | 14 | Output voltage is too high |
| `ERR_WARN_LIM_HIGH` | 15 | Upper warning limit exceeded |
| `ERR_TEMP_LOW` | 20 | Temperature is too low |
| `ERR_NEG_IN_VOLT` | 21 | Negative input voltage detected |
| `ERR_IN_CURR_SMALL` | 22 | Input current is too low |
| `ERR_NEG_IN_CURR` | 23 | Negative input current detected |
| `ERR_NEG_IN_CURR_SENSOR_NOT_CONN` | 24 | Negative input current or disconnected sensor detected |
| `ERR_OUT_VOLT_SMALL` | 25 | Output voltage is too low |
| `ERR_WARN_LIM_LOW` | 26 | Lower warning limit exceeded |
| `ERR_SENSOR_NOT_CONN` | 30 | Sensor is not connected |
| `ERR_LOW_LEVEL_EXCEEDS_VOLT_LIMIT` | 31 | Low signal level exceeds the voltage limit |
| `ERR_OUT_OVERLOAD` | 40 | Output overload detected |
| `ERR_VIRT_NULL` | 50 | Virtual value is null |
| `ERR_VIRT_ALARM` | 51 | Virtual value is in an alarm state |
| `ERR_VIRT_STALE` | 52 | Virtual value is stale |
| `ERR_VIRT_DOWN` | 53 | Virtual source is unavailable |
| `ERR_VIRT_FAULT` | 54 | Virtual source reports a fault |
| `ERR_VIRT_DISABLED` | 55 | Virtual source is disabled |

The following numeric values were previously assigned and must not be reused:

| Value | Previous name |
|---:|---|
| 1 | `ERR_OVERRANGE` |
| 2 | `ERR_UNDERRANGE` |
| 3 | `ERR_BROKEN_WIRE` |
| 4 | `ERR_SHORT_CIRCUIT` |

Error codes do not replace `PointStatus`.

A client should normally evaluate:

```text
status
error_code
process value
```

together.

### `OverrideAction`

`OverrideAction` selects the runtime operation applied to an override.

| Name | Value | Description |
|---|---:|---|
| `OVERRIDE_ACTION_NONE` | 0 | No override operation |
| `OVERRIDE_ACTION_START` | 1 | Starts an override |
| `OVERRIDE_ACTION_STOP` | 2 | Stops an override |
| `OVERRIDE_ACTION_CANCEL` | 3 | Cancels an override |

Starting an override normally also requires:

- An interface-specific override value
- An override duration, where supported

Examples of interface-specific override values include:

```text
digital_override_val
analog_override_val
counter_override_val
```

`OVERRIDE_ACTION_STOP` and `OVERRIDE_ACTION_CANCEL` are separate protocol values. Their detailed behavioral distinction is defined by the device's override implementation.

### `TestStatus`

`TestStatus` represents the state of an I/O wiring test.

| Name | Value | Description |
|---|---:|---|
| `TEST_DONT_CHANGE` | 0 | Retains the current test status when used in a request |
| `TEST_OPEN` | 1 | The wiring test is open or pending |
| `TEST_PASSED` | 2 | The wiring test passed |
| `TEST_FAILED` | 3 | The wiring test failed |
| `TEST_BLOCKED` | 4 | The wiring test is blocked |
| `TEST_SKIPPED` | 5 | The wiring test was skipped |

`test_status` is separate from:

- `status`, which represents the general point state
- `error_code`, which represents the current I/O error

When returned in a response, `test_status` represents the current test state reported by the device.

### `Setting`

`Setting` is used by generic setting messages that contain a selector and an associated value.

| Name | Value | Description |
|---|---:|---|
| `SET_DONT_CHANGE` | 0 | Retains the currently configured value |
| `SET_DEFAULT` | 1 | Applies the device-specific default value |
| `SET_VAL` | 2 | Applies the value supplied in the associated value field |

Generic setting messages normally contain:

```text
setting
val
```

The associated value is meaningful when `setting` is `SET_VAL`.

Depending on the message type, the value may instead be stored in another field such as:

```text
vals
name
```

### `BooleanSetting`

`BooleanSetting` combines the requested setting operation and Boolean value in a single enum.

| Name | Value | Description |
|---|---:|---|
| `BOOLEAN_SET_DONT_CHANGE` | 0 | Retains the currently configured Boolean value |
| `BOOLEAN_SET_DEFAULT` | 1 | Applies the device-specific default Boolean value |
| `BOOLEAN_SET_FALSE` | 2 | Explicitly sets the value to `false` |
| `BOOLEAN_SET_TRUE` | 3 | Explicitly sets the value to `true` |

Unlike the generic setting messages, `BooleanSetting` is not a nested message.

For example:

```proto
message UiCfgReq {
    BooleanSetting enabled = 1;
}
```

A request that explicitly enables the channel uses:

```text
enabled = BOOLEAN_SET_TRUE
```

The corresponding Protobuf JSON representation is:

```json
{
  "enabled": "BOOLEAN_SET_TRUE"
}
```

It is not represented as a nested object with separate `setting` and `val` fields.

## Messages

### `SupportPoint`

`SupportPoint` represents one X/Y point used by a line or characteristic curve.

| Field | Type | Tag | Description |
|---|---|---:|---|
| `x` | `float` | 1 | Input coordinate |
| `y` | `float` | 2 | Output coordinate |

Example:

```json
{
  "x": 0,
  "y": 0
}
```

### `Line`

`Line` contains up to two supporting points.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `vals` | Repeated `SupportPoint` | 1 | Maximum 2 entries | Supporting points of the line |

A two-point calibration normally uses exactly two supporting points:

```text
(X1, Y1)
(X2, Y2)
```

The interface-specific documentation defines whether fewer than two points are accepted.

Example:

```json
{
  "vals": [
    {
      "x": 0,
      "y": 0
    },
    {
      "x": 10,
      "y": 10.2
    }
  ]
}
```

### `Curve`

`Curve` represents a named characteristic curve containing up to 20 supporting points.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `name` | `string` | 2 | Maximum 64 characters | Name of the characteristic curve |
| `vals` | Repeated `SupportPoint` | 3 | Maximum 20 entries | Supporting points of the characteristic curve |

Field number 1 was previously used and must not be reused.

The interface-specific documentation defines:

- The minimum number of supporting points
- The required ordering of supporting points
- Interpolation behavior
- Behavior outside the configured point range

Example:

```json
{
  "name": "Temperature conversion",
  "vals": [
    {
      "x": 0,
      "y": 0
    },
    {
      "x": 5,
      "y": 50
    },
    {
      "x": 10,
      "y": 100
    }
  ]
}
```

### `String16Setting`

`String16Setting` configures a string with a maximum length of 16 characters.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `setting` | `Setting` | 1 | None | Selects the requested setting operation |
| `val` | `string` | 2 | Maximum 16 characters | String value applied when `setting` is `SET_VAL` |

Example:

```json
{
  "setting": "SET_VAL",
  "val": "Room 1"
}
```

### `String64Setting`

`String64Setting` configures a string with a maximum length of 64 characters.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `setting` | `Setting` | 1 | None | Selects the requested setting operation |
| `val` | `string` | 2 | Maximum 64 characters | String value applied when `setting` is `SET_VAL` |

Example:

```json
{
  "setting": "SET_VAL",
  "val": "Room temperature input"
}
```

### `Uint64Setting`

`Uint64Setting` configures an unsigned 64-bit integer.

| Field | Type | Tag | Description |
|---|---|---:|---|
| `setting` | `Setting` | 1 | Selects the requested setting operation |
| `val` | `uint64` | 2 | Unsigned 64-bit value applied when `setting` is `SET_VAL` |

Protobuf JSON represents 64-bit integer values as JSON strings.

Example:

```json
{
  "setting": "SET_VAL",
  "val": "1786032000"
}
```

### `Uint32Setting`

`Uint32Setting` configures an unsigned 32-bit integer.

| Field | Type | Tag | Description |
|---|---|---:|---|
| `setting` | `Setting` | 1 | Selects the requested setting operation |
| `val` | `uint32` | 2 | Unsigned 32-bit value applied when `setting` is `SET_VAL` |

Example:

```json
{
  "setting": "SET_VAL",
  "val": 900
}
```

### `FloatSetting`

`FloatSetting` configures a floating-point value.

| Field | Type | Tag | Description |
|---|---|---:|---|
| `setting` | `Setting` | 1 | Selects the requested setting operation |
| `val` | `float` | 2 | Floating-point value applied when `setting` is `SET_VAL` |

Example:

```json
{
  "setting": "SET_VAL",
  "val": 1.5
}
```

Special floating-point values such as `NaN` may have an interface-specific meaning.

### `LineSetting`

`LineSetting` configures two-point line or calibration data.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `setting` | `Setting` | 1 | None | Selects the requested setting operation |
| `vals` | Repeated `SupportPoint` | 2 | Maximum 2 entries | Supporting points applied when `setting` is `SET_VAL` |

Example:

```json
{
  "setting": "SET_VAL",
  "vals": [
    {
      "x": 0,
      "y": 0
    },
    {
      "x": 10,
      "y": 10.2
    }
  ]
}
```

The interface-specific documentation defines whether exactly two supporting points are required.

### `CurveSetting`

`CurveSetting` configures a named characteristic curve containing up to 20 supporting points.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `setting` | `Setting` | 1 | None | Selects the requested setting operation |
| `name` | `string` | 3 | Maximum 64 characters | Name of the characteristic curve |
| `vals` | Repeated `SupportPoint` | 4 | Maximum 20 entries | Supporting points applied when `setting` is `SET_VAL` |

Field number 2 was previously used and must not be reused.

Example:

```json
{
  "setting": "SET_VAL",
  "name": "Linear conversion",
  "vals": [
    {
      "x": 0,
      "y": 0
    },
    {
      "x": 10,
      "y": 100
    }
  ]
}
```

The interface-specific documentation defines:

- The minimum number of supporting points
- Required point ordering
- Interpolation behavior
- Behavior outside the configured range

### `IoErrorCodeSetting`

`IoErrorCodeSetting` configures an `IoErrorCode` value.

| Field | Type | Tag | Description |
|---|---|---:|---|
| `setting` | `Setting` | 1 | Selects the requested setting operation |
| `val` | `IoErrorCode` | 2 | I/O error code applied when `setting` is `SET_VAL` |

This setting type may be used for virtual-channel error injection or other interface-specific diagnostic operations.

Example:

```json
{
  "setting": "SET_VAL",
  "val": "ERR_VIRT_ALARM"
}
```

The interface-specific documentation defines where error-code configuration is permitted.

### `Ipv4Setting`

`Ipv4Setting` configures a fixed-length IPv4 address.

| Field | Type | Tag | Constraint | Description |
|---|---|---:|---|---|
| `setting` | `Setting` | 1 | None | Selects the requested setting operation |
| `val` | `bytes` | 2 | Fixed length of 4 bytes | IPv4 address applied when `setting` is `SET_VAL` |

At the binary Protobuf level, `val` contains the four IPv4 address octets.

For example:

```text
192.168.1.10
```

is represented by the byte sequence:

```text
C0 A8 01 0A
```

In standard Protobuf JSON, `bytes` values are Base64-encoded strings.

Example:

```json
{
  "setting": "SET_VAL",
  "val": "wKgBCg=="
}
```

The Base64 value must be decoded before it is formatted as a dotted-decimal IPv4 address.

## Setting behavior summary

### Boolean fields versus generic value fields

Fields of type `BooleanSetting` directly contain the requested operation and value in a single enum value:

```json
{
  "enabled": "BOOLEAN_SET_TRUE"
}
```

They do not use a nested object.

Message-based setting types use a nested object containing `setting` and the associated value field or fields:

```json
{
  "name": {
    "setting": "SET_VAL",
    "val": "Room temperature"
  }
}
```

### Operation selection

| Selector | Behavior |
|---|---|
| `SET_DONT_CHANGE` / `BOOLEAN_SET_DONT_CHANGE` | The currently configured value is retained. Associated value fields must not be interpreted as active values. |
| `SET_DEFAULT` / `BOOLEAN_SET_DEFAULT` | The device-specific default value is applied. The Protobuf schema does not define the actual default value. |
| `SET_VAL` / `BOOLEAN_SET_FALSE` / `BOOLEAN_SET_TRUE` | The associated value is applied. For generic setting messages, the relevant value fields (`val`, `vals`, or `name` and `vals`) depend on the setting message type. |

## Constraint summary

| Type | Constraint |
|---|---|
| `Line.vals` | Maximum 2 supporting points |
| `Curve.name` | Maximum 64 characters |
| `Curve.vals` | Maximum 20 supporting points |
| `String16Setting.val` | Maximum 16 characters |
| `String64Setting.val` | Maximum 64 characters |
| `LineSetting.vals` | Maximum 2 supporting points |
| `CurveSetting.name` | Maximum 64 characters |
| `CurveSetting.vals` | Maximum 20 supporting points |
| `Ipv4Setting.val` | Fixed length of 4 bytes |

The nanopb constraints define maximum encoded capacities. They do not necessarily define all semantic validation rules.

For example:

- `Line.vals` allows up to two entries at the schema level.
- An interface may require exactly two entries for a valid calibration.
- `Curve.vals` allows up to 20 entries at the schema level.
- An interface may require at least two entries and ascending X coordinates.

## Reserved values and fields

The following enum values and message fields were previously used and must not be reused.

### `IoErrorCode`

| Value | Previous enum name |
|---:|---|
| 1 | `ERR_OVERRANGE` |
| 2 | `ERR_UNDERRANGE` |
| 3 | `ERR_BROKEN_WIRE` |
| 4 | `ERR_SHORT_CIRCUIT` |

### `Curve`

| Field number | Previous field |
|---:|---|
| 1 | Repeated `SupportPoint vals` using fixed-count encoding |

### `CurveSetting`

| Field number | Previous field |
|---:|---|
| 2 | Repeated `SupportPoint vals` using fixed-count encoding |

These values and field numbers must remain unused to preserve wire compatibility with earlier schema versions.

## JSON serialization notes

When Protocol Buffers messages are converted to JSON:

- Enum values are normally represented by their symbolic names.
- `uint64` values are represented as JSON strings.
- `bytes` fields are represented as Base64-encoded strings.
- Repeated fields are represented as JSON arrays.
- Protobuf snake-case field names are normally converted to lower camel case.
- Omitted scalar fields may be interpreted as their Protobuf default values.
- A numeric enum value of `0` is the default value when an enum field is omitted.

The JSON representation of `val` depends on the concrete setting message:

| Message | JSON value representation |
|---|---|
| `String16Setting` | JSON string |
| `String64Setting` | JSON string |
| `Uint32Setting` | JSON number |
| `Uint64Setting` | JSON string |
| `FloatSetting` | JSON number |
| `IoErrorCodeSetting` | Enum name or numeric enum value |
| `Ipv4Setting` | Base64-encoded JSON string |

Representative examples of the corresponding JSON objects are already listed in the message sections above (for instance in `Uint32Setting`, `Uint64Setting`, and `Ipv4Setting`).

## Complete shared-type overview

| Category | Type | Purpose |
|---|---|---|
| Enum | `PointStatus` | General state of an I/O point |
| Enum | `IoErrorCode` | Detailed I/O error or diagnostic state |
| Enum | `OverrideAction` | Override runtime operation |
| Enum | `TestStatus` | I/O wiring-test state |
| Enum | `Setting` | Operation selector for generic setting messages |
| Enum | `BooleanSetting` | Combined Boolean setting operation and value |
| Message | `SupportPoint` | X/Y supporting point |
| Message | `Line` | Line or two-point calibration data |
| Message | `Curve` | Named characteristic curve |
| Message | `String16Setting` | Setting for a string with up to 16 characters |
| Message | `String64Setting` | Setting for a string with up to 64 characters |
| Message | `Uint64Setting` | Setting for an unsigned 64-bit value |
| Message | `Uint32Setting` | Setting for an unsigned 32-bit value |
| Message | `FloatSetting` | Setting for a floating-point value |
| Message | `LineSetting` | Setting for line or calibration data |
| Message | `CurveSetting` | Setting for characteristic-curve data |
| Message | `IoErrorCodeSetting` | Setting for an I/O error code |
| Message | `Ipv4Setting` | Setting for a fixed-length IPv4 address |
