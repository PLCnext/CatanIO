# CatanIO: UOI, DOI, and UV API Documentation

## Overview

This API defines the configuration and runtime control of Universal I/O (UOI) channels and the related DOI and UV channel subsets via **CoAP**.

DOI and UV channels reuse the UOI Protobuf message definitions and therefore share the same request and response structures with UOI.

Communication is handled using **Protocol Buffers** messages.

The following concrete operating modes are supported:

- Digital Output (`UOI_TYPE_DO`)
- Digital Input (`UOI_TYPE_DI`)
- Counter Input (`UOI_TYPE_COUNTER`)
- Analog Voltage Output (`UOI_TYPE_AO_VOLTAGE`)

## Dependencies

- `shared.proto`: Enums, settings, and shared helper structures
- `uoi.proto`: UOI-specific messages and enums

## Channel subsets

### DOI channels

DOI channels are a digital subset of UOI and support the following operating modes:

- Digital Output
- Digital Input
- Counter Input

DOI channels use the UOI Protobuf message definitions but do not expose fields that are exclusive to Analog Output processing or to signal-conditioning options that are not applicable to purely digital signals.

#### Configuration fields not exposed by DOI

The following UOI configuration fields are not applicable to DOI channels:

```text
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
```

#### Runtime request fields not exposed by DOI

The following UOI runtime request fields are not applicable to DOI channels:

```text
analog_set_val
analog_override_val
counter_override_val
error_code
```

#### Runtime write model

The DOI runtime write model therefore consists of:

```text
digital_set_val
counter_init_val
init_on_time_s
override_action
override_duration_ms
digital_override_val
```

`counter_init_val` and `init_on_time_s` are part of the DOI write model because the corresponding process values (`counter_val` and `on_time_s`) are available in the applicable DOI modes, and the underlying `UoiReq` message defines the fields.

### UV channels

UV channels are an output-oriented subset of UOI and support the following operating modes:

- Digital Output
- Analog Voltage Output

UV channels are used as virtual channels written by a connected display. They use the UOI Protobuf message definitions but expose only a subset of the fields defined by `UoiCfgReq` and `UoiReq`.

Because UV channels are virtual, the virtual-channel error injection field `error_code` is applicable to UV runtime requests.

#### Configuration fields exposed by UV

The UV configuration write model consists of:

```text
enabled
type
name
override_enabled
override_cancelable
local_setup_enabled
digital_default_val
use_default_val_on_start
use_default_val_on_failure
cov_increment
analog_default_val
configured
```

#### Configuration fields not exposed by UV

The following UOI configuration fields are not applicable to UV channels:

```text
inverted
two_point_calibration
characteristic_curve
unit
counter_scale
min_send_time_ms
max_send_time_ms
minimum
maximum
lower_fault_level
upper_fault_level
debounced
false_text
true_text
```

#### Runtime request fields not exposed by UV

The following UOI runtime request fields are not applicable to UV channels:

```text
counter_override_val
```

#### Runtime write model

The UV runtime write model consists of:

```text
digital_set_val
analog_set_val
counter_init_val
init_on_time_s
override_action
override_duration_ms
digital_override_val
analog_override_val
error_code
```

`counter_init_val` and `init_on_time_s` are part of the UV write model because Digital Output mode provides the corresponding process values (`counter_val` as the DO switching counter and `on_time_s` as the accumulated true-state time).

`error_code` is included because UV channels are virtual and support I/O error injection through `IoErrorCodeSetting`.

## API endpoints

### UOI API endpoints

| Method | Endpoint | Request | Response | Description |
|---|---|---|---|---|
| `GET` | `/uoi/<channel>/cfg` | None | `UoiCfgResp` | Reads the effective UOI channel configuration |
| `PATCH` | `/uoi/<channel>/cfg` | `UoiCfgReq` | `UoiCfgResp` | Changes the UOI channel configuration |
| `GET` | `/uoi/<channel>` | None | `UoiResp` | Reads the current UOI runtime values |
| `PATCH` | `/uoi/<channel>` | `UoiReq` | `UoiResp` | Writes set values, initialization values, or override commands |

### DOI API endpoints

DOI endpoints use the UOI Protobuf message definitions.

| Method | Endpoint | Request | Response | Description |
|---|---|---|---|---|
| `GET` | `/doi/<channel>/cfg` | None | `UoiCfgResp` | Reads the effective DOI channel configuration |
| `PATCH` | `/doi/<channel>/cfg` | `UoiCfgReq` | `UoiCfgResp` | Changes the DOI channel configuration |
| `GET` | `/doi/<channel>` | None | `UoiResp` | Reads the current DOI runtime values |
| `PATCH` | `/doi/<channel>` | `UoiReq` | `UoiResp` | Writes set values, initialization values, or override commands |

### UV API endpoints

UV endpoints use the UOI Protobuf message definitions.

| Method | Endpoint | Request | Response | Description |
|---|---|---|---|---|
| `GET` | `/uv/<channel>/cfg` | None | `UoiCfgResp` | Reads the effective UV channel configuration |
| `PATCH` | `/uv/<channel>/cfg` | `UoiCfgReq` | `UoiCfgResp` | Changes the UV channel configuration |
| `GET` | `/uv/<channel>` | None | `UoiResp` | Reads the current UV runtime values |
| `PATCH` | `/uv/<channel>` | `UoiReq` | `UoiResp` | Writes set values, initialization values, or override commands |

## Channels on Catan modules

| Module | Channel type | Channel numbers |
|---|---|---|
| Catan C1 | UV | 1-8 |
|  | DOI | 9-12 |
|  | UOI | 13-14 |
| Catan DOI8 UOI8 UI8 | UV | 1-8 |
|  | DOI | 9-16 |
|  | UOI | 17-24 |
| Catan DALI MM4 DOI8 | DOI | 1-8 |
|  | UV | 1-8 |

Identical channel numbers may appear for different channel types on the same module because each channel type has its own addressing space (for example, DOI 1-8 and UV 1-8 refer to different physical channels).

## Operating modes

The following concrete operating modes are defined by `UoiType`:

| Enum value | Numeric value | Description |
|---|---:|---|
| `UOI_TYPE_DO` | 2 | Digital Output |
| `UOI_TYPE_DI` | 3 | Digital Input |
| `UOI_TYPE_COUNTER` | 4 | Counter Input |
| `UOI_TYPE_AO_VOLTAGE` | 5 | Analog Voltage Output |

## Runtime value semantics

### Counter and on-time values

The UOI supports counters in three operating modes. The meaning of `counter_val` depends on the active mode.

| Mode | Meaning of `counter_val` |
|---|---|
| DO | Switching counter that counts rising edges of the effective Digital Output signal |
| DI | Switching counter that counts rising edges of the effective Digital Input signal |
| Counter | Primary process value that counts incoming hardware pulses or rising edges |

A transition from `false` to `true` increments the switching counter:

```text
false -> true    counter_val += 1
true  -> false   counter_val remains unchanged
```

A complete inactive-active-inactive switching cycle therefore increments the switching counter once.

The meaning of `on_time_s` is shared by the two digital modes:

| Mode | Meaning of `on_time_s` |
|---|---|
| DO | Accumulated time for which the effective Digital Output signal was `true` |
| DI | Accumulated time for which the effective Digital Input signal was `true` |
| Counter | Not available |
| AO | Not available |

The accumulated values can be initialized using the following runtime request fields:

| Runtime request field | Applicable modes | Effect |
|---|---|---|
| `counter_init_val` | DO, DI, Counter | Sets the mode-specific `counter_val` to the requested initial value |
| `init_on_time_s` | DO, DI | Sets `on_time_s` to the requested initial value in seconds |

Initialization and override are different operations:

- Initialization changes an accumulated process value.
- An override temporarily replaces the effective signal or process value.

## Signal-path overview

### Digital Output

The Digital Output path processes a digital set value.

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
```

The normal value of the DO path is provided by `digital_set_val`.

Depending on the configuration and operating state, this value may be replaced by:

- `digital_default_val` when the device starts
- `digital_default_val` when communication with the control unit is interrupted
- `digital_override_val` while an override is active

The resulting value is processed by the inversion stage.

The effective DO signal provides:

| Runtime value | Meaning |
|---|---|
| `digital_set_val` | Requested normal Digital Output value |
| `digital_val` | Effective Digital Output value after default, override, and inversion processing |
| `on_time_s` | Accumulated time for which `digital_val` was `true` |
| `counter_val` | Switching counter that counts rising edges of `digital_val` |

The accumulated values can be initialized using:

```text
counter_init_val
init_on_time_s
```

### Digital Input

The Digital Input path processes a digital value received from the hardware driver.

```text
Hardware digital input
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
```

The normal value of the DI path is received from the hardware input.

While an override is active, `digital_override_val` replaces the hardware input value. The resulting value is then processed by the inversion stage.

The effective DI signal provides:

| Runtime value | Meaning |
|---|---|
| `digital_val` | Effective Digital Input value after override and inversion processing |
| `on_time_s` | Accumulated time for which `digital_val` was `true` |
| `counter_val` | Switching counter that counts rising edges of `digital_val` |

The accumulated values can be initialized using:

```text
counter_init_val
init_on_time_s
```

In DI mode:

- `digital_val` remains the primary process value.
- `on_time_s` and `counter_val` are secondary values derived from the effective digital signal.

The DI switching counter does not change the operating mode to the dedicated Counter mode.

### Counter Input

The Counter Input path processes incoming hardware pulses.

```text
Hardware pulse input
        |
        v
 rising-edge counter
        |
        +-- counter_override_val
        |
        +------------------------------> counter_val
        |
        v
   counter_scale
        |
        +------------------------------> analog_val
```

The dedicated Counter mode provides two representations:

| Runtime value | Meaning |
|---|---|
| `counter_val` | Primary Counter representation |
| `analog_val` | Scaled analog representation of the Counter value |

The relationship between the two representations is conceptually:

```text
analog_val = counter_val * counter_scale
```

The implementation may internally use the inverse relationship when converting a scaled analog representation back to a Counter value:

```text
counter_val = analog_val / counter_scale
```

Rounding, overflow, division-by-zero, and precision behavior are implementation-specific.

The Counter value can be initialized using:

```text
counter_init_val
```

The Counter value can be overridden using:

```text
counter_override_val
```

`analog_override_val` is not valid in Counter mode. It is assigned to Analog Output mode.

Counter mode does not provide:

```text
digital_val
on_time_s
```

### Analog Output

The Analog Output path processes an analog set value.

```text
analog_set_val
       |
       +-- analog_default_val
       |
       +-- analog_override_val
       |
       v
   limit check
       |
       v
    limit cap
       |
       +------------------------------> analog_val
       |
       v
two-point calibration
       |
       v
characteristic curve
       |
       v
 error handling
       |
       v
 actuator_val
       |
       v
Hardware driver
```

The normal value of the AO path is provided by `analog_set_val`.

Depending on the configuration and operating state, this value may be replaced by:

- `analog_default_val` when the device starts
- `analog_default_val` when communication with the control unit is interrupted
- `analog_override_val` while an override is active

The value is then processed by:

1. Limit checking
2. Lower and upper capping
3. Optional two-point calibration
4. Optional characteristic curve conversion
5. Error handling

The Analog Output path provides three different runtime values:

| Runtime value | Meaning |
|---|---|
| `analog_set_val` | Requested normal Analog Output value |
| `analog_val` | Logical Analog Output value after default, override, and limit processing |
| `actuator_val` | Physical output signal in volts after calibration and characteristic curve processing |

These values represent different processing stages and must not be treated as aliases.

The hardware driver receives the resulting `actuator_val`.

## Validity legend

| Symbol | Meaning |
|---|---|
| `✓` | Valid and meaningful for the operating mode |
| `-` | Not valid for the operating mode |
| `G` | Mode-independent general field |

`G` means that a field is independent of the selected concrete operating mode. Additional restrictions, such as applicability to virtual channels only, may still apply.

## Configuration data points

Configuration data points are defined in:

- `UoiCfgReq` for configuration changes
- `UoiCfgResp` for returned configuration values

In `UoiCfgReq`, fields represented by a `*Setting` type contain a requested configuration operation or value.

In `UoiCfgResp`, the corresponding fields contain the effective configuration returned by the device.

The mode validity is identical for corresponding fields in both messages.

### General configuration

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `enabled` | 1 | G | G | G | G | Indicates or configures whether the UOI channel is active; an inactive channel does not access its hardware signals |
| `type` | 2 | G | G | G | G | Contains or selects the concrete `UoiType` and therefore the hardware and signal-processing mode |
| `name` | 3 | G | G | G | G | User-defined channel name with a maximum length of 64 characters |
| `override_enabled` | 4 | G | G | G | G | Enables override control through the API or a connected display |
| `override_cancelable` | 5 | G | G | G | G | Allows an active override to be canceled through the API or a connected display |
| `configured` | 25 | G | G | G | G | Indicates or configures whether the UOI channel is marked as configured |
| `local_setup_enabled` | 26 | G | G | G | G | Allows a connected display to configure the I/O channel |
| `test_status` | 27 | G | G | G | G | Contains or configures the `TestStatus` of the I/O wiring test stored in the I/O device |

### Default-value configuration

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `digital_default_val` | 6 | ✓ | - | - | - | Preferred digital output value used when the configured startup or communication-failure behavior is active |
| `analog_default_val` | 7 | - | - | - | ✓ | Preferred analog output value used when the configured startup or communication-failure behavior is active |
| `use_default_val_on_start` | 8 | ✓ | - | - | ✓ | Selects whether the output uses its configured default value when the device starts; otherwise, the last value is retained |
| `use_default_val_on_failure` | 9 | ✓ | - | - | ✓ | Selects whether the output uses its configured default value when communication with the control unit is interrupted; otherwise, the last value is retained |

### Signal-processing configuration

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `inverted` | 10 | ✓ | ✓ | - | - | Selects whether the effective digital signal corresponds to the logical signal or its logical inverse |
| `two_point_calibration` | 11 | - | - | - | ✓ | Optional linear signal correction defined by two points `(X1, Y1)` and `(X2, Y2)` |
| `characteristic_curve` | 12 | - | - | - | ✓ | Optional tabular signal-conversion curve containing between two and 20 supporting points |
| `unit` | 13 | - | ✓ | - | ✓ | Free text specifying the engineering unit, with a maximum length of 16 characters |
| `counter_scale` | 14 | - | - | ✓ | - | Scale factor used to derive `analog_val` from the Counter value |
| `debounced` | 22 | - | ✓ | - | - | Enables or configures debouncing of the Digital Input signal |

#### `inverted`

For DO:

- `false`: The effective output signal corresponds to the data point value.
- `true`: The effective output signal corresponds to the inverse of the data point value.

For DI:

- `false`: The effective input value corresponds to the logical hardware signal.
- `true`: The effective input value corresponds to the inverse of the logical hardware signal.

The DI switching counter and on-time value are derived from the effective value after inversion.

#### `two_point_calibration`

Two-point calibration is an optional linear correction defined by:

```text
(X1, Y1)
(X2, Y2)
```

The two points define a linear function used to convert the signal value and compensate for systematic deviations.

#### `characteristic_curve`

The characteristic curve is an optional tabular conversion for analog signal processing.

It must contain:

- At least two supporting points
- No more than 20 supporting points

Supporting points are processed in ascending order.

A curve containing fewer than two supporting points is invalid.

#### `counter_scale`

`counter_scale` defines the relationship between the Counter value and its scaled analog representation:

```text
analog_val = counter_val * counter_scale
```

It applies only to `UOI_TYPE_COUNTER`.

#### `unit`

`unit` is free text used to specify the engineering unit of the data point.

The maximum length is 16 characters.

### Change-of-value and transmission configuration

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `cov_increment` | 15 | - | ✓ | - | ✓ | Minimum change in the process value that causes a change notification |
| `min_send_time_ms` | 16 | - | ✓ | - | - | Minimum interval between two change notifications in milliseconds |
| `max_send_time_ms` | 17 | - | ✓ | - | - | Maximum configured transmission interval in milliseconds |

The exact interpretation of `cov_increment` for a Boolean DI value is implementation-specific.

### Analog limits and fault handling

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `minimum` | 18 | - | - | - | ✓ | Lower capping limit; smaller logical AO values are limited to this value |
| `maximum` | 19 | - | - | - | ✓ | Upper capping limit; larger logical AO values are limited to this value |
| `lower_fault_level` | 20 | - | - | - | ✓ | Lower fault threshold; smaller values cause the data point to enter an error state |
| `upper_fault_level` | 21 | - | - | - | ✓ | Upper fault threshold; larger values cause the data point to enter an error state |

`minimum` and `maximum` are capping limits:

- Values below `minimum` are limited to `minimum`.
- Values above `maximum` are limited to `maximum`.

Crossing a capping limit does not by itself imply a fault.

The fault thresholds belong to the AO error-handling stage:

- Values below `lower_fault_level` cause an error state.
- Values above `upper_fault_level` cause an error state.

An inactive analog limit may be represented as `NaN`.

### Digital display configuration

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `false_text` | 23 | ✓ | ✓ | - | - | Display text for the digital `false` state; if empty, `false` is displayed; maximum length is 16 characters |
| `true_text` | 24 | ✓ | ✓ | - | - | Display text for the digital `true` state; if empty, `true` is displayed; maximum length is 16 characters |

The display texts apply to Boolean process values and do not change the actual Boolean value.

## Runtime request data points

Runtime request data points are defined in `UoiReq`.

They are written using:

```text
PATCH /uoi/<channel>
```

The same message definition is used for the corresponding DOI and UV endpoints.

### Set values and initialization values

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `digital_set_val` | 1 | ✓ | - | - | - | Specifies the requested normal Digital Output value |
| `analog_set_val` | 2 | - | - | - | ✓ | Specifies the requested normal Analog Output value |
| `counter_init_val` | 3 | ✓ | ✓ | ✓ | - | Sets the mode-specific Counter value to the requested initial value |
| `init_on_time_s` | 4 | ✓ | ✓ | - | - | Sets the accumulated on-time value to the requested initial value in seconds |

The meaning of `counter_init_val` depends on the active mode:

| Mode | Effect |
|---|---|
| DO | Initializes the switching counter derived from rising edges of `digital_val` |
| DI | Initializes the switching counter derived from rising edges of `digital_val` |
| Counter | Initializes the primary hardware-pulse counter |
| AO | Not valid |

The meaning of `init_on_time_s` depends on the active mode:

| Mode | Effect |
|---|---|
| DO | Initializes the accumulated time for which the effective DO value was `true` |
| DI | Initializes the accumulated time for which the effective DI value was `true` |
| Counter | Not valid |
| AO | Not valid |

### Override control

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `override_action` | 5 | G | G | G | G | Specifies the `OverrideAction` to apply to the UOI channel |
| `override_duration_ms` | 6 | G | G | G | G | Specifies the requested override duration in milliseconds; `0` means unlimited |
| `digital_override_val` | 7 | ✓ | ✓ | - | - | Specifies the digital value used while a digital override is active |
| `analog_override_val` | 8 | - | - | - | ✓ | Specifies the Analog Output value used while an override is active |
| `counter_override_val` | 9 | - | - | ✓ | - | Specifies the Counter value used while a Counter override is active |

The override value must match the active mode:

| Mode | Override value | Effect |
|---|---|---|
| DO | `digital_override_val` | Temporarily replaces the normal Digital Output value |
| DI | `digital_override_val` | Temporarily replaces the hardware Digital Input value |
| Counter | `counter_override_val` | Temporarily replaces the effective Counter value |
| AO | `analog_override_val` | Temporarily replaces the normal Analog Output value |

### Virtual-channel error injection

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `error_code` | 10 | G | G | G | G | Injects or changes the I/O error code of a virtual UOI channel; applicable only to virtual channels |

`error_code` is mode-independent but is applicable only to virtual channels.

It is not a normal physical-channel set value.

## Runtime response data points

Runtime response data points are defined in `UoiResp`.

They are returned using:

```text
GET /uoi/<channel>
```

The same message definition is used for the corresponding DOI and UV endpoints.

### General response data

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `type` | 1 | G | G | G | G | Contains the active concrete `UoiType` |
| `status` | 2 | G | G | G | G | Contains the current general point state represented by `PointStatus` |
| `error_code` | 3 | G | G | G | G | Contains the current I/O error represented by `IoErrorCode` |
| `override_remaining_ms` | 4 | G | G | G | G | Contains the remaining duration of the active override in milliseconds |
| `test_status` | 12 | G | G | G | G | Contains the current `TestStatus` of the I/O wiring test stored in the device |

The runtime status fields have different meanings:

| Data point | Meaning |
|---|---|
| `status` | General point state represented by `PointStatus` |
| `error_code` | Current I/O error represented by `IoErrorCode` |
| `test_status` | State of the I/O wiring test represented by `TestStatus` |
| `override_remaining_ms` | Remaining runtime of an active override in milliseconds |

### Mode-dependent response data

| Data point | Field number | DO | DI | Counter | AO | Description |
|---|---:|:---:|:---:|:---:|:---:|---|
| `digital_val` | 5 | ✓ | ✓ | - | - | Contains the effective digital value after the applicable default, override, and inversion processing |
| `analog_val` | 6 | - | - | ✓ | ✓ | Contains the scaled Counter representation or the logical Analog Output value, depending on the active mode |
| `counter_val` | 7 | ✓ | ✓ | ✓ | - | Contains the DO or DI rising-edge switching counter, or the primary Counter mode value |
| `on_time_s` | 8 | ✓ | ✓ | - | - | Contains the accumulated time in seconds for which the effective digital value was `true` |
| `digital_set_val` | 9 | ✓ | - | - | - | Contains the currently requested normal Digital Output value |
| `analog_set_val` | 10 | - | - | - | ✓ | Contains the currently requested normal Analog Output value |
| `actuator_val` | 11 | - | - | - | ✓ | Contains the physical output signal in volts after calibration and characteristic curve processing |

## Complete validity matrix: UOI

### UOI configuration fields

| Data point | DO | DI | Counter | AO | Description |
|---|:---:|:---:|:---:|:---:|---|
| `enabled` | G | G | G | G | Indicates or configures whether the UOI channel is active |
| `type` | G | G | G | G | Contains or selects the concrete `UoiType` |
| `name` | G | G | G | G | User-defined channel name with a maximum length of 64 characters |
| `override_enabled` | G | G | G | G | Enables override control through the API or a connected display |
| `override_cancelable` | G | G | G | G | Allows an active override to be canceled through the API or a connected display |
| `digital_default_val` | ✓ | - | - | - | Preferred DO value used by the configured default behavior |
| `analog_default_val` | - | - | - | ✓ | Preferred AO value used by the configured default behavior |
| `use_default_val_on_start` | ✓ | - | - | ✓ | Selects the default or retained value at startup |
| `use_default_val_on_failure` | ✓ | - | - | ✓ | Selects the default or retained value after communication loss |
| `inverted` | ✓ | ✓ | - | - | Selects the logical or inverted digital signal |
| `two_point_calibration` | - | - | - | ✓ | Optional linear correction defined by two points |
| `characteristic_curve` | - | - | - | ✓ | Optional conversion curve with two to 20 supporting points |
| `unit` | - | ✓ | - | ✓ | Free-text engineering unit with a maximum length of 16 characters |
| `counter_scale` | - | - | ✓ | - | Scale factor for the Counter `analog_val` representation |
| `cov_increment` | - | ✓ | - | ✓ | Minimum change in the process value that causes a change notification |
| `min_send_time_ms` | - | ✓ | - | - | Minimum change-notification interval in milliseconds |
| `max_send_time_ms` | - | ✓ | - | - | Maximum configured transmission interval in milliseconds |
| `minimum` | - | - | - | ✓ | Lower capping limit |
| `maximum` | - | - | - | ✓ | Upper capping limit |
| `lower_fault_level` | - | - | - | ✓ | Lower error threshold |
| `upper_fault_level` | - | - | - | ✓ | Upper error threshold |
| `debounced` | - | ✓ | - | - | Enables or configures Digital Input debouncing |
| `false_text` | ✓ | ✓ | - | - | Display text for `false` with a maximum length of 16 characters |
| `true_text` | ✓ | ✓ | - | - | Display text for `true` with a maximum length of 16 characters |
| `configured` | G | G | G | G | Indicates or configures whether the channel is configured |
| `local_setup_enabled` | G | G | G | G | Enables local setup using a connected display |
| `test_status` | G | G | G | G | Contains or configures the `TestStatus` of the I/O wiring test |

### UOI runtime request fields

| Data point | DO | DI | Counter | AO | Description |
|---|:---:|:---:|:---:|:---:|---|
| `digital_set_val` | ✓ | - | - | - | Requested normal Digital Output value |
| `analog_set_val` | - | - | - | ✓ | Requested normal Analog Output value |
| `counter_init_val` | ✓ | ✓ | ✓ | - | Initializes the mode-specific Counter value |
| `init_on_time_s` | ✓ | ✓ | - | - | Initializes the accumulated on-time value in seconds |
| `override_action` | G | G | G | G | `OverrideAction` applied to the channel |
| `override_duration_ms` | G | G | G | G | Requested override duration in milliseconds; `0` means unlimited |
| `digital_override_val` | ✓ | ✓ | - | - | Digital value used during an override |
| `analog_override_val` | - | - | - | ✓ | Analog Output value used during an override |
| `counter_override_val` | - | - | ✓ | - | Counter value used during an override |
| `error_code` | G | G | G | G | Injects or changes the I/O error code of a virtual channel; applicable only to virtual channels |

### UOI runtime response fields

| Data point | DO | DI | Counter | AO | Description |
|---|:---:|:---:|:---:|:---:|---|
| `type` | G | G | G | G | Active concrete `UoiType` |
| `status` | G | G | G | G | Current `PointStatus` |
| `error_code` | G | G | G | G | Current `IoErrorCode` |
| `override_remaining_ms` | G | G | G | G | Remaining override duration in milliseconds |
| `digital_val` | ✓ | ✓ | - | - | Effective digital value |
| `analog_val` | - | - | ✓ | ✓ | Scaled Counter value or logical AO value |
| `counter_val` | ✓ | ✓ | ✓ | - | DO or DI switching counter, or primary Counter mode value |
| `on_time_s` | ✓ | ✓ | - | - | Accumulated true-state time in seconds |
| `digital_set_val` | ✓ | - | - | - | Requested normal DO value |
| `analog_set_val` | - | - | - | ✓ | Requested normal AO value |
| `actuator_val` | - | - | - | ✓ | Physical AO signal in volts |
| `test_status` | G | G | G | G | Current I/O wiring test status |

## JSON serialization notes

Depending on the Protobuf-to-JSON conversion settings, a response may include fields with default values even if those fields are not meaningful for the active operating mode.

Consumers must use the active concrete `type` together with the validity matrix to determine which mode-dependent fields are meaningful.

Fields that are not valid for the active mode must not be interpreted as active process values.

## Shared enums and message types

For more information, see [`shared/README.md`](../shared/README.md).

- `PointStatus`
  - Used in: `UoiResp.status`
- `IoErrorCode`
  - Used in: `UoiResp.error_code`, `IoErrorCodeSetting.val`
- `OverrideAction`
  - Used in: `UoiReq.override_action`
- `TestStatus`
  - Used in: `UoiCfgReq.test_status`, `UoiCfgResp.test_status`, `UoiResp.test_status`
- `Setting`
  - Used by the corresponding `*Setting` messages

## Example runtime response

### Digital Input response: `GET /uoi/<channel>`

```json
{
  "type": "UOI_TYPE_DI",
  "status": "STATUS_OK",
  "errorCode": "ERR_OK",
  "overrideRemainingMs": 0,
  "digitalVal": false,
  "counterVal": "0",
  "onTimeS": 0,
  "testStatus": "TEST_OPEN"
}
```

In DI mode:

- `digitalVal` is the primary process value.
- `counterVal` is the switching counter derived from rising edges of `digitalVal`.
- `onTimeS` is the accumulated time for which `digitalVal` was `true`.

## Example configuration response

### Digital Input configuration: `GET /uoi/<channel>/cfg`

```json
{
  "enabled": true,
  "type": "UOI_TYPE_DI",
  "name": "Digital input 1",
  "overrideEnabled": true,
  "overrideCancelable": false,
  "inverted": false,
  "unit": "",
  "covIncrement": 0,
  "minSendTimeMs": 100,
  "maxSendTimeMs": 0,
  "debounced": true,
  "falseText": "Off",
  "trueText": "On",
  "configured": true,
  "localSetupEnabled": false,
  "testStatus": "TEST_OPEN"
}
```
