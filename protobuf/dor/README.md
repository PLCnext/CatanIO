# CatanIO: DOR API Documentation

## Overview

This API defines the configuration and runtime control of dedicated relay output (DOR) channels via **CoAP**.

Communication is handled using **Protocol Buffers** messages.

A DOR channel is a dedicated Digital Output implemented as a relay output. Unlike DOI, UOI, and UI channels, a DOR channel does not provide multiple selectable operating modes.

The DOR interface therefore has:

- No configurable `type` field
- No mode-selection enum
- No Digital Input mode
- No Counter Input mode
- No Analog Input or Analog Output mode

A DOR channel always operates as:

```text
Digital Output -> Relay hardware
```

## Dependencies

- `shared.proto`: Enums, settings, and shared helper structures
- `dor.proto`: DOR-specific messages

## API endpoints

The DOR API uses the following channel-specific CoAP endpoints and Protobuf messages:

| Method | Endpoint | Request | Response | Description |
|---|---|---|---|---|
| `GET` | `/dor/<channel>/cfg` | None | `DorCfgResp` | Reads the effective DOR channel configuration |
| `PATCH` | `/dor/<channel>/cfg` | `DorCfgReq` | `DorCfgResp` | Changes the DOR channel configuration |
| `GET` | `/dor/<channel>` | None | `DorResp` | Reads the current DOR runtime values |
| `PATCH` | `/dor/<channel>` | `DorReq` | `DorResp` | Writes set values, initialization values, or override commands |

The configuration endpoint reads or changes persistent channel configuration.

The runtime endpoint reads process values or writes set values, initialization values, and override commands.

## Channels on Catan modules

| Module | Channel type | Channel numbers |
|---|---|---|
| Catan DOR6 UI8 | DOR | 1-6 |

## Fixed operating mode

A DOR channel supports exactly one functional operating mode:

| Processing mode | Hardware function | Description |
|---|---|---|
| Digital Output | Relay Output | Controls a relay using a Boolean set value |

A DOR channel is identified as a relay output by:

- The `/dor/` endpoint
- Its physical channel type
- The `DorCfgReq`, `DorCfgResp`, `DorReq`, and `DorResp` message types

DOR requests do not require mode resolution before field validation.

## Runtime value semantics

The DOR interface provides one requested output value, one effective output value, and two accumulated runtime values:

| Runtime value | Meaning |
|---|---|
| `digital_set_val` | Currently requested normal Digital Output value before default, override, and inversion processing |
| `digital_val` | Effective relay output value after default selection, override selection, and inversion processing |
| `counter_val` | Switching counter that counts transitions of `digital_val` from `false` to `true` |
| `on_time_s` | Accumulated time in seconds for which `digital_val` was `true` |

`digital_set_val` and `digital_val` represent different processing stages and must not be treated as aliases.

A transition from `false` to `true` increments the switching counter:

```text
false -> true    counter_val += 1
true  -> false   counter_val remains unchanged
```

A complete inactive-active-inactive switching cycle therefore increments `counter_val` once.

`counter_val` represents transitions of the effective logical output value. It does not represent verified mechanical relay operations.

The accumulated values can be initialized using the following runtime request fields:

| Runtime request field | Effect |
|---|---|
| `counter_init_val` | Sets `counter_val` to the requested initial value |
| `init_on_time_s` | Sets `on_time_s` to the requested initial value in seconds |

Initialization and override are different operations:

- Initialization changes an accumulated runtime value.
- An override temporarily replaces the selected normal Digital Output value.
- Initialization does not force the relay output.
- An override does not directly reset the counter or on-time value.

## Signal-path overview

The DOR signal path processes a Boolean set value.

```text
digital_set_val
        |
        +-- digital_default_val
        |
        +-- digital_override_val
        |
        v
     inverted
        |
        +------------------------------> digital_val
        |
        +-- true-state duration -------> on_time_s
        |
        +-- rising-edge counter -------> counter_val
        |
        v
 DOR hardware driver
        |
        v
      Relay
```

The normal logical output value is supplied by:

```text
digital_set_val
```

Depending on the configuration and operating state, this value may be replaced by:

- `digital_default_val` when the device starts
- `digital_default_val` when communication with the control unit is interrupted
- `digital_override_val` while an override is active

The selected logical value is then processed by the inversion stage.

The resulting `digital_val`:

- Is returned as the effective output value
- Is passed to the DOR hardware driver
- Is used for on-time accumulation
- Is used by the rising-edge switching counter

The runtime response provides:

```text
digital_set_val
digital_val
counter_val
on_time_s
```

## Requested and effective output values

`digital_set_val` and `digital_val` represent different processing stages:

| Data point | Processing stage |
|---|---|
| `digital_set_val` | Normal logical output value requested by the client |
| `digital_val` | Effective output value after default selection, override selection, and inversion processing |

The values may differ because of:

- Startup default-value processing
- Communication-failure default-value processing
- An active override
- Inversion

### Examples

| Scenario | `digital_set_val` | `digital_override_val` | Override active | `inverted` | `digital_val` |
|---|:---:|:---:|:---:|:---:|:---:|
| Inversion only | `true` | – | no | `true` | `false` |
| Override only | `false` | `true` | yes | `false` | `true` |
| Override and inversion | `false` | `true` | yes | `true` | `false` |

A client must not use `digital_set_val` as a substitute for the effective output state.

The effective output state is represented by:

```text
digital_val
```

## Startup and failure behavior

The DOR interface supports a configurable logical default value:

```text
digital_default_val
```

The following fields determine when that value is used:

```text
use_default_val_on_start
use_default_val_on_failure
```

The selected default value is processed by the inversion stage before it becomes `digital_val`.

### Startup behavior

| `use_default_val_on_start` | Startup behavior |
|:---:|---|
| `false` | The relay retains or restores its last value according to the device implementation |
| `true` | The relay uses `digital_default_val` |

### Communication-failure behavior

| `use_default_val_on_failure` | Communication-failure behavior |
|:---:|---|
| `false` | The relay retains its last effective value |
| `true` | The relay uses `digital_default_val` |

A communication failure refers to interrupted communication with the control unit.

### Default value

| `digital_default_val` | Logical meaning before inversion |
|:---:|---|
| `false` | Inactive or low |
| `true` | Active or high |

`digital_default_val` is a logical value before inversion.

The resulting effective value is determined conceptually as follows:

```text
digital_val = inverted
    ? NOT selected_logical_value
    : selected_logical_value
```

The selected logical value may originate from:

```text
digital_set_val
digital_default_val
digital_override_val
```

## Validity legend

| Symbol | Meaning |
|---|---|
| `✓` | Valid and meaningful for DOR |
| `-` | Not exposed by DOR |
| `G` | General DOR field |

Because DOR has only one fixed operating mode, the validity tables do not require separate mode columns.

## Configuration data points

Configuration data points are defined in:

- `DorCfgReq` for configuration changes
- `DorCfgResp` for returned configuration values

In `DorCfgReq`, fields represented by a `*Setting` type contain a requested configuration operation or value.

In `DorCfgResp`, the corresponding fields contain the effective configuration returned by the device.

### DOR configuration fields

| Data point | Field number | Validity | Description |
|---|---:|:---:|---|
| `enabled` | 1 | G | Indicates or configures whether the DOR channel is active; an inactive channel does not access or control its relay signal |
| `name` | 2 | G | User-defined channel name with a maximum length of 64 characters |
| `override_enabled` | 3 | G | Enables override control through the API or a connected display |
| `override_cancelable` | 4 | G | Allows an active override to be canceled through the API or a connected display |
| `inverted` | 5 | ✓ | Selects whether the selected logical output value is inverted before it is exposed as `digital_val` and passed to the relay hardware driver |
| `use_default_val_on_start` | 6 | ✓ | Selects whether the relay uses `digital_default_val` when the device starts |
| `use_default_val_on_failure` | 7 | ✓ | Selects whether the relay uses `digital_default_val` when communication with the control unit is interrupted |
| `digital_default_val` | 8 | ✓ | Logical relay output value selected by the configured startup or communication-failure behavior before inversion |
| `false_text` | 9 | ✓ | Display text for the digital `false` state; if empty, `false` is displayed; maximum length is 16 characters |
| `true_text` | 10 | ✓ | Display text for the digital `true` state; if empty, `true` is displayed; maximum length is 16 characters |
| `configured` | 11 | G | Indicates or configures whether the DOR channel is marked as configured |
| `local_setup_enabled` | 12 | G | Allows a connected display to configure the I/O channel |
| `test_status` | 13 | G | Contains or configures the `TestStatus` of the I/O wiring test stored in the I/O device |

## Configuration semantics

### `enabled`

`enabled` and `configured` represent different channel states:

- `enabled` controls whether the channel is active.
- `configured` indicates whether the channel is marked as configured.

A configured DOR channel may still be disabled.

### `override_enabled`

`override_enabled` enables override control through the API or a connected display.

It does not itself start an override.

The runtime override operation is controlled through:

```text
override_action
override_duration_ms
digital_override_val
```

### `override_cancelable`

`override_cancelable` determines whether an active override may be canceled.

The field does not indicate whether an override is currently active.

The remaining duration of an active override is reported through:

```text
override_remaining_ms
```

### `inverted`

`inverted` is applied after the normal, default, or override value has been selected.

| `inverted` | Behavior |
|:---:|---|
| `false` | `digital_val` corresponds to the selected logical output value |
| `true` | `digital_val` corresponds to the inverse of the selected logical output value |

Conceptually:

```text
inverted = false:
    digital_val = selected logical value

inverted = true:
    digital_val = NOT selected logical value
```

The selected logical value may originate from:

- `digital_set_val`
- `digital_default_val`
- `digital_override_val`

### Display texts

`false_text` and `true_text` affect only the displayed representation of the Boolean output value.

They do not change:

- `digital_set_val`
- `digital_val`
- The relay hardware output

### `local_setup_enabled`

`local_setup_enabled` controls whether the DOR channel may be configured locally using a connected display.

It is separate from `override_enabled`:

- `local_setup_enabled` controls local configuration.
- `override_enabled` controls override availability.

### `test_status`

`test_status` represents the state of the I/O wiring test stored in the I/O device.

It is separate from:

- `status`, which represents the general runtime point state
- `error_code`, which represents the current I/O error

## Runtime request data points

Runtime request data points are defined in `DorReq`.

They are written using:

```text
PATCH /dor/<channel>
```

### DOR runtime request fields

| Data point | Field number | Validity | Description |
|---|---:|:---:|---|
| `digital_set_val` | 1 | ✓ | Specifies the requested normal logical Digital Output value before default, override, and inversion processing |
| `override_action` | 2 | G | Specifies the `OverrideAction` to apply to the DOR channel |
| `digital_override_val` | 3 | ✓ | Specifies the logical value selected while an override is active; the selected value is subsequently processed by the inversion stage |
| `override_duration_ms` | 4 | G | Specifies the requested override duration in milliseconds; `0` means unlimited |
| `counter_init_val` | 5 | ✓ | Sets the DOR switching counter to the requested initial value |
| `init_on_time_s` | 6 | ✓ | Sets the accumulated DOR on-time value to the requested initial value in seconds |

### Normal set value

`digital_set_val` specifies the normal Boolean output request.

It is processed before:

- Default-value selection
- Override selection
- Inversion

`digital_set_val` is not guaranteed to equal `digital_val`.

Conceptually:

```text
normal request:
    digital_set_val

selected logical value:
    digital_set_val
    OR digital_default_val
    OR digital_override_val

effective output value:
    selected logical value processed by inverted
```

### Accumulated-value initialization

#### `counter_init_val`

`counter_init_val` sets the switching counter to the requested initial value.

After initialization, the switching counter continues counting transitions of `digital_val` from `false` to `true`.

Initializing the counter does not:

- Switch the relay
- Create an output edge
- Start an override
- Change `digital_set_val`

#### `init_on_time_s`

`init_on_time_s` sets the accumulated on-time value to the requested initial value in seconds.

After initialization, `on_time_s` continues accumulating while `digital_val` is `true`.

Initializing the on-time value does not:

- Force the relay to `true`
- Change `digital_set_val`
- Start an override

### Override control

The DOR override uses:

```text
override_action
override_duration_ms
digital_override_val
```

`digital_override_val` is selected before inversion. The resulting effective output value therefore depends on `inverted`.

#### Override actions

`override_action` uses the shared `OverrideAction` enum.

| Enum value | Numeric value | Meaning |
|---|---:|---|
| `OVERRIDE_ACTION_NONE` | 0 | No override action |
| `OVERRIDE_ACTION_START` | 1 | Start an override |
| `OVERRIDE_ACTION_STOP` | 2 | Stop an override |
| `OVERRIDE_ACTION_CANCEL` | 3 | Cancel an override |

`OVERRIDE_ACTION_STOP` and `OVERRIDE_ACTION_CANCEL` represent distinct override operations. Their detailed behavioral distinction is defined by the shared override specification.

The requested override operation is supplied through:

```text
override_action
```

The logical override value is supplied through:

```text
digital_override_val
```

The requested duration is supplied through:

```text
override_duration_ms
```

A duration of:

```text
override_duration_ms = 0
```

means that the override duration is unlimited.

The runtime response reports the remaining override duration through:

```text
override_remaining_ms
```

An effective output transition caused by an override may affect:

- `digital_val`
- `counter_val`
- `on_time_s`

The counter and on-time values are derived from `digital_val`, not solely from `digital_set_val`.

## Runtime response data points

Runtime response data points are defined in `DorResp`.

They are returned using:

```text
GET /dor/<channel>
```

A `DorResp` is also returned after a successful runtime request:

```text
PATCH /dor/<channel>
```

### DOR runtime response fields

| Data point | Field number | Validity | Description |
|---|---:|:---:|---|
| `status` | 1 | G | Contains the current general point state represented by `PointStatus` |
| `error_code` | 2 | G | Contains the current I/O error represented by `IoErrorCode` |
| `override_remaining_ms` | 3 | G | Contains the remaining duration of the active override in milliseconds |
| `digital_val` | 4 | ✓ | Contains the effective relay output value after default selection, override selection, and inversion processing |
| `counter_val` | 5 | ✓ | Contains the switching counter that counts transitions of `digital_val` from `false` to `true` |
| `on_time_s` | 6 | ✓ | Contains the accumulated time in seconds for which `digital_val` was `true` |
| `digital_set_val` | 7 | ✓ | Contains the currently requested normal Digital Output value |
| `test_status` | 8 | G | Contains the current `TestStatus` of the I/O wiring test |

`DorResp` does not contain a `type` field because every DOR channel has the fixed Relay Output operating mode.

### Runtime status semantics

The runtime status fields have different meanings:

| Data point | Meaning |
|---|---|
| `status` | General point state represented by `PointStatus` |
| `error_code` | Current I/O error represented by `IoErrorCode` |
| `test_status` | State of the I/O wiring test represented by `TestStatus` |
| `override_remaining_ms` | Remaining runtime of an active override in milliseconds |

These fields must not be treated as aliases.

### Output response semantics

| Data point | Meaning |
|---|---|
| `digital_set_val` | Currently requested normal relay value before default, override, and inversion processing |
| `digital_val` | Effective relay output value after default, override, and inversion processing |
| `counter_val` | Switching counter that counts transitions of `digital_val` from `false` to `true` |
| `on_time_s` | Accumulated time in seconds for which `digital_val` was `true` |

## Complete DOR validity matrix

### DOR configuration fields

| Data point | Validity | Description |
|---|:---:|---|
| `enabled` | G | Indicates or configures whether the DOR channel is active |
| `name` | G | User-defined channel name with a maximum length of 64 characters |
| `override_enabled` | G | Enables override control through the API or a connected display |
| `override_cancelable` | G | Allows an active override to be canceled through the API or a connected display |
| `inverted` | ✓ | Inverts the selected logical output value before it becomes `digital_val` |
| `use_default_val_on_start` | ✓ | Selects the configured default value at startup |
| `use_default_val_on_failure` | ✓ | Selects the configured default value after communication loss |
| `digital_default_val` | ✓ | Logical relay value used by the configured default behavior before inversion |
| `false_text` | ✓ | Display text for `false` with a maximum length of 16 characters |
| `true_text` | ✓ | Display text for `true` with a maximum length of 16 characters |
| `configured` | G | Indicates or configures whether the channel is configured |
| `local_setup_enabled` | G | Enables local setup using a connected display |
| `test_status` | G | Contains or configures the `TestStatus` of the I/O wiring test |

### DOR runtime request fields

| Data point | Validity | Description |
|---|:---:|---|
| `digital_set_val` | ✓ | Requested normal logical relay output value |
| `override_action` | G | Override operation applied to the channel |
| `digital_override_val` | ✓ | Logical value selected during an active override before inversion |
| `override_duration_ms` | G | Requested override duration in milliseconds; `0` means unlimited |
| `counter_init_val` | ✓ | Initializes the switching counter |
| `init_on_time_s` | ✓ | Initializes the accumulated on-time value in seconds |

### DOR runtime response fields

| Data point | Validity | Description |
|---|:---:|---|
| `status` | G | Current `PointStatus` |
| `error_code` | G | Current `IoErrorCode` |
| `override_remaining_ms` | G | Remaining override duration in milliseconds |
| `digital_val` | ✓ | Effective relay output value after default, override, and inversion processing |
| `counter_val` | ✓ | Rising-edge switching counter derived from `digital_val` |
| `on_time_s` | ✓ | Accumulated time in seconds for which `digital_val` was `true` |
| `digital_set_val` | ✓ | Currently requested normal relay output value |
| `test_status` | G | Current I/O wiring test status |

## Fields not exposed by DOR

Fields from other Catan I/O interfaces are not automatically valid for DOR.

Clients must only send fields defined by the corresponding DOR Protobuf message.

### Configuration fields

The DOR configuration interface does not expose:

```text
type
open_contact
analog_default_val
two_point_calibration
characteristic_curve
unit
counter_scale
cov_increment
min_send_time_ms
max_send_time_ms
minimum
maximum
lower_fault_level
upper_fault_level
debounced
```

### Runtime request fields

The DOR runtime request interface does not expose:

```text
analog_set_val
analog_override_val
counter_override_val
error_code
```

### Runtime response fields

The DOR runtime response interface does not expose:

```text
type
analog_val
analog_set_val
actuator_val
sensor_val
```

The presence of similarly named fields in UI, UOI, or DOI messages does not make them valid for DOR.

## Shared enums and message types

For more information, see [`shared/README.md`](../shared/README.md).

- `PointStatus`
  - Used in: `DorResp.status`
- `IoErrorCode`
  - Used in: `DorResp.error_code`
- `OverrideAction`
  - Used in: `DorReq.override_action`
- `TestStatus`
  - Used in: `DorCfgReq.test_status`, `DorCfgResp.test_status`, `DorResp.test_status`
- `Setting`
  - Used by the corresponding `*Setting` messages

## JSON serialization notes

Depending on the Protobuf-to-JSON conversion settings, responses may include fields with their default values.

The DOR interface has only one fixed operating mode. Therefore, every field defined in `DorResp` has a fixed DOR-specific meaning.

Numeric 64-bit integer fields, such as `counter_val`, may be represented as JSON strings depending on the Protobuf JSON conversion.

For example:

```json
{
  "counterVal": "42"
}
```

## Example runtime response

### Relay Output response: `GET /dor/<channel>`

```json
{
  "status": "STATUS_OK",
  "errorCode": "ERR_OK",
  "overrideRemainingMs": 0,
  "digitalVal": false,
  "counterVal": "0",
  "onTimeS": 0,
  "digitalSetVal": false,
  "testStatus": "TEST_OPEN"
}
```

In this example:

- `digitalSetVal` is the requested normal output value.
- `digitalVal` is the effective output value.
- `counterVal` contains the switching counter.
- `onTimeS` contains the accumulated true-state time.
- No override is active because `overrideRemainingMs` is `0`.

## Example configuration response

### Relay Output configuration: `GET /dor/<channel>/cfg`

```json
{
  "enabled": true,
  "name": "Relay output 1",
  "overrideEnabled": true,
  "overrideCancelable": true,
  "inverted": false,
  "useDefaultValOnStart": true,
  "useDefaultValOnFailure": true,
  "digitalDefaultVal": false,
  "falseText": "Off",
  "trueText": "On",
  "configured": true,
  "localSetupEnabled": false,
  "testStatus": "TEST_OPEN"
}
```
