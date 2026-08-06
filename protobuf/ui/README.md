# CatanIO: UI API Documentation

## Overview

This API defines the configuration and runtime control of Universal Input (UI) channels via **CoAP**.

Communication is handled using **Protocol Buffers** messages.

Universal Input channels support the following functional operating modes:

- Digital Input
- Counter Input
- Analog Input

## Dependencies

- `shared.proto`: Enums, settings, and shared helper structures
- `ui.proto`: UI-specific messages and enums

## API endpoints

The UI API uses the following CoAP endpoints and Protobuf messages:

| Method | Endpoint | Request | Response | Description |
|---|---|---|---|---|
| `GET` | `/ui/<channel>/cfg` | None | `UiCfgResp` | Reads the effective UI channel configuration |
| `PATCH` | `/ui/<channel>/cfg` | `UiCfgReq` | `UiCfgResp` | Changes the UI channel configuration |
| `GET` | `/ui/<channel>` | None | `UiResp` | Reads the current UI runtime values |
| `PATCH` | `/ui/<channel>` | `UiReq` | `UiResp` | Writes initialization values and override commands |

The configuration endpoint reads or changes persistent channel configuration.

The runtime endpoint reads process values or writes initialization and override values.

## Channels on Catan modules

| Module | Channel type | Channel numbers |
|---|---|---|
| Catan C1 | UI | 1-8 |
| Catan DOR UI8 | UI | 1-8 |
| Catan DOI8 UOI8 UI8 | UI | 1-8 |

## Operating modes

The concrete `UiType` values are grouped into three functional processing modes:

| Processing mode | Protobuf types | Description |
|---|---|---|
| Digital Input | `UI_TYPE_DI` | Digital Input with an underlying switching counter and accumulated on-time value |
| Counter Input | `UI_TYPE_COUNTER` | Counter Input with an underlying digital input, primary counter value, and scaled counter representation |
| Analog Input | All concrete `UI_TYPE_AI_*` types | Analog Input with sensor-level and processed application values |

### Analog Input types

The following concrete types use the common Analog Input processing path:

```text
UI_TYPE_AI_VOLTAGE
UI_TYPE_AI_CURRENT_0_20
UI_TYPE_AI_CURRENT_4_20
UI_TYPE_AI_RESISTOR_10K
UI_TYPE_AI_RESISTOR_180K
UI_TYPE_AI_TEMPERATURE_PT1000_DIN
UI_TYPE_AI_TEMPERATURE_PT1000_SAMA
UI_TYPE_AI_TEMPERATURE_NI1000_DIN
UI_TYPE_AI_TEMPERATURE_NI1000_SAMA
UI_TYPE_AI_TEMPERATURE_NI1000_LG
UI_TYPE_AI_TEMPERATURE_NTC_10K
UI_TYPE_AI_TEMPERATURE_NTC_10K_PRE
UI_TYPE_AI_TEMPERATURE_NTC_20K
```

The selected concrete `UiType` controls:

- The hardware measuring range
- Hardware-specific input processing
- Type conversion
- Sensor-specific error handling

### Context-dependent types

The following values do not directly identify a concrete signal-processing mode:

| Protobuf type | Numeric value | Meaning |
|---|---:|---|
| `UI_TYPE_DONT_CHANGE` | 0 | Retain the currently configured concrete UI type |
| `UI_TYPE_DEFAULT` | 1 | Use the device-specific default UI type; `UI_TYPE_AI_VOLTAGE` is the preferred default |

Before validating mode-dependent fields, the effective concrete type must be resolved:

- `UI_TYPE_DONT_CHANGE` resolves to the currently active UI type.
- `UI_TYPE_DEFAULT` resolves to the device-specific default UI type.

## Runtime value semantics

### Counter and on-time values

UI channels provide counter values in Digital Input and Counter Input modes. The meaning of `counter_val` depends on the active mode.

| Processing mode | Meaning of `counter_val` |
|---|---|
| Digital Input | Secondary switching counter that counts rising edges of the effective Digital Input value |
| Counter Input | Primary counter value that counts configured edges of the hardware input signal |

In Digital Input mode, a transition from `false` to `true` increments the switching counter:

```text
false -> true    counter_val += 1
true  -> false   counter_val remains unchanged
```

A complete inactive-active-inactive switching cycle therefore increments the switching counter once.

In Counter Input mode, the counted hardware edge depends on `inverted`:

| `inverted` | Counted hardware edge |
|:---:|---|
| `false` | Rising edge of the logical input signal |
| `true` | Falling edge of the logical input signal |

The meaning of `on_time_s` depends on the processing mode:

| Processing mode | Meaning of `on_time_s` |
|---|---|
| Digital Input | Accumulated time for which the effective Digital Input value was `true` |
| Counter Input | Not available |
| Analog Input | Not available |

The accumulated values can be initialized using the following runtime request fields:

| Runtime request field | Applicable modes | Effect |
|---|---|---|
| `counter_init_val` | Digital Input, Counter Input | Sets the mode-specific `counter_val` to the requested initial value |
| `init_on_time_s` | Digital Input | Sets `on_time_s` to the requested initial value in seconds |

Initialization and override are different operations:

- Initialization changes an accumulated process value.
- An override temporarily replaces an effective process value.

## Signal-path overview

### Digital Input

The Digital Input path processes a digital value received from the hardware driver.

```text
Hardware digital input
        |
        v
 open-contact mode
        |
        v
     inverted
        |
        +-- digital_override_val
        |
        +------------------------------> digital_val
        |
        +-- true-state duration -------> on_time_s
        |
        +-- rising-edge counter -------> counter_val
```

The hardware-specific input processing includes:

- The channel enabled state
- Digital-input mode
- Open-contact configuration
- Inversion

While an override is active, `digital_override_val` replaces the processed digital input value after open-contact and inversion processing.

The effective Digital Input signal provides:

| Runtime value | Meaning |
|---|---|
| `digital_val` | Effective Digital Input value after open-contact, inversion, and override processing |
| `counter_val` | Secondary switching counter that counts rising edges of `digital_val` |
| `on_time_s` | Accumulated time in seconds for which `digital_val` was `true` |

The accumulated values can be initialized using:

```text
counter_init_val
init_on_time_s
```

In Digital Input mode:

- `digital_val` is the primary process value.
- `counter_val` and `on_time_s` are secondary values derived from the effective digital signal.

The Digital Input switching counter does not change the operating mode to the dedicated Counter Input mode.

### Counter Input

The Counter Input path processes a logical hardware input and its accumulated edge count.

```text
Hardware digital input
        |
        v
 open-contact mode
        |
        v
     inverted
        |
        +------------------------------> digital_val
        |
        v
 configured-edge counter
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

Counter Input mode provides three related runtime values:

| Runtime value | Meaning |
|---|---|
| `digital_val` | Effective logical input signal used by the Counter Input path |
| `counter_val` | Primary integer counter representation |
| `analog_val` | Scaled analog representation of the counter value |

The counted edge depends on `inverted`:

- When `inverted` is `false`, the counter increments on rising edges of the logical input signal.
- When `inverted` is `true`, the counter increments on falling edges of the logical input signal.

The counter value can be initialized using:

```text
counter_init_val
```

The counter value can be overridden using:

```text
counter_override_val
```

The relationship between the two counter representations is conceptually:

```text
analog_val = counter_val * counter_scale
```

The implementation may internally use the inverse relationship when converting a scaled analog representation back to a counter value:

```text
counter_val = analog_val / counter_scale
```

`analog_override_val` is not valid in Counter Input mode. It is assigned to Analog Input mode.

Counter Input mode does not provide:

```text
on_time_s
sensor_val
```

Rounding, integer overflow, division-by-zero, negative-scale, conversion, and floating-point precision behavior are implementation-specific.

### Analog Input

All concrete `UI_TYPE_AI_*` types use the common Analog Input path.

```text
Hardware input
       |
       v
Analog hardware mode
       |
       v
temperature error handling
       |
       v
type conversion
       |
       v
error handling
       |
       +------------------------------> sensor_val
       |
       v
characteristic_curve
       |
       v
two_point_calibration
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
```

Temperature error handling applies to temperature input types.

Other Analog Input types use the common downstream processing path without temperature-specific conversion.

The Analog Input path provides two different runtime values:

| Runtime value | Meaning |
|---|---|
| `sensor_val` | Measured input value after hardware-specific type conversion and error handling, expressed in the unit associated with the selected Analog Input type |
| `analog_val` | Fully processed application value after characteristic curve, two-point calibration, override, and analog limit processing |

The following functions do not affect `sensor_val`:

- Characteristic curve
- Two-point calibration
- Analog limits
- Analog override

The two values represent different processing stages and must not be treated as aliases.

A client should use:

- `sensor_val` when the sensor-level measurement is required.
- `analog_val` when the fully processed application value is required.

## Validity legend

| Symbol | Meaning |
|---|---|
| `✓` | Valid and meaningful for the processing mode |
| `-` | Not valid for the processing mode |
| `G` | Mode-independent general field |

`G` means that a field is independent of the selected concrete operating mode. Additional channel-specific restrictions may still apply.

## Configuration data points

Configuration data points are defined in:

- `UiCfgReq` for configuration changes
- `UiCfgResp` for returned configuration values

In `UiCfgReq`, fields represented by a `*Setting` type contain a requested configuration operation or value.

In `UiCfgResp`, the corresponding fields contain the effective configuration returned by the device.

The mode validity is identical for corresponding fields in both messages.

### General configuration

| Data point | Field number | DI | Counter | Analog | Description |
|---|---:|:---:|:---:|:---:|---|
| `enabled` | 1 | G | G | G | Indicates or configures whether the UI channel is active; an inactive channel does not access its hardware signals |
| `type` | 2 | G | G | G | Contains or selects the concrete `UiType` and therefore the hardware and signal-processing mode |
| `name` | 3 | G | G | G | User-defined channel name with a maximum length of 64 characters |
| `override_enabled` | 4 | G | G | G | Enables override control through the API or a connected display |
| `override_cancelable` | 5 | G | G | G | Allows an active override to be canceled through the API or a connected display |
| `configured` | 22 | G | G | G | Indicates or configures whether the UI channel is marked as configured |
| `local_setup_enabled` | 23 | G | G | G | Allows a connected display to configure the I/O channel |
| `test_status` | 24 | G | G | G | Contains or configures the `TestStatus` of the I/O wiring test stored in the I/O device |

`enabled` and `configured` describe different channel states:

- `enabled` controls whether the channel is active and accesses its signals.
- `configured` indicates whether the channel is marked as configured.

`override_enabled` enables override control. It does not itself start an override.

`override_cancelable` determines whether an active override may be canceled.

`local_setup_enabled` controls whether the channel may be configured locally using a connected display.

`test_status` describes the I/O wiring test. It is separate from the runtime `status` and `error_code` fields.

### Digital and Counter Input configuration

Digital Input and Counter Input modes share hardware-level digital-input configuration.

| Data point | Field number | DI | Counter | Description |
|---|---:|:---:|:---:|---|
| `inverted` | 6 | ✓ | ✓ | Inverts the Digital Input value or selects the counted edge in Counter Input mode |
| `open_contact` | 7 | ✓ | ✓ | Selects voltage-level input or internally supplied open-contact input wiring |
| `unit` | 10 | ✓ | ✓ | Free text specifying the engineering unit, with a maximum length of 16 characters |
| `counter_scale` | 11 | - | ✓ | Scale factor used to derive `analog_val` from the counter value |
| `cov_increment` | 12 | ✓ | ✓ | Minimum change in the process value that causes a change notification |
| `min_send_time_ms` | 13 | ✓ | ✓ | Minimum interval between two change notifications in milliseconds |
| `max_send_time_ms` | 14 | ✓ | ✓ | Maximum configured transmission interval in milliseconds |
| `false_text` | 20 | ✓ | ✓ | Display text for the digital `false` state; if empty, `false` is displayed; maximum length is 16 characters |
| `true_text` | 21 | ✓ | ✓ | Display text for the digital `true` state; if empty, `true` is displayed; maximum length is 16 characters |

#### `inverted`

For Digital Input:

- `false`: `digital_val` corresponds to the logical hardware input.
- `true`: `digital_val` corresponds to the inverse of the logical hardware input.

For Counter Input:

- `false`: The counter increments on each rising edge of the logical input signal.
- `true`: The counter increments on each falling edge of the logical input signal.

#### `open_contact`

When `open_contact` is `false`, the input evaluates the externally applied voltage level.

When `open_contact` is `true`, the input supplies a voltage that can be connected to signal ground through a potential-free contact:

- A closed contact is evaluated as high.
- An open contact is evaluated as low.

#### `counter_scale`

`counter_scale` applies only to Counter Input mode and defines the scale factor used to derive `analog_val` from `counter_val`.

Conceptually:

```text
analog_val = counter_val * counter_scale
```

It is not used for the secondary switching counter in Digital Input mode.

#### Display texts

`false_text` and `true_text` describe the underlying digital input value.

They affect only its displayed representation and do not change the Boolean signal.

### Analog Input configuration

| Data point | Field number | Description |
|---|---:|---|
| `two_point_calibration` | 8 | Optional linear signal correction defined by two points `(X1, Y1)` and `(X2, Y2)` |
| `characteristic_curve` | 9 | Optional tabular signal-conversion curve containing between two and 20 supporting points |
| `unit` | 10 | Free text specifying the engineering unit, with a maximum length of 16 characters |
| `cov_increment` | 12 | Minimum change in the process value that causes a change notification |
| `min_send_time_ms` | 13 | Minimum interval between two change notifications in milliseconds |
| `max_send_time_ms` | 14 | Maximum configured transmission interval in milliseconds |
| `minimum` | 15 | Lower capping limit; smaller processed values are limited to this value |
| `maximum` | 16 | Upper capping limit; larger processed values are limited to this value |
| `lower_fault_level` | 17 | Lower fault threshold; smaller values cause the data point to enter an error state |
| `upper_fault_level` | 18 | Upper fault threshold; larger values cause the data point to enter an error state |

The Analog Input configuration is shared by all concrete `UI_TYPE_AI_*` types.

The selected concrete Analog Input type controls the hardware measuring range and type-conversion behavior.

#### `characteristic_curve`

The characteristic curve is an optional nonlinear conversion represented by a table.

It must contain:

- At least two supporting points
- No more than 20 supporting points

The supporting points are processed in ascending order.

A curve containing fewer than two supporting points is invalid.

#### `two_point_calibration`

Two-point calibration is an optional linear correction defined by:

```text
(X1, Y1)
(X2, Y2)
```

The two points define a linear function used to compensate for systematic deviations.

#### Analog limits

`minimum` and `maximum` are capping limits:

- Values below `minimum` are limited to `minimum`.
- Values above `maximum` are limited to `maximum`.

Crossing a capping limit does not by itself imply a fault.

`lower_fault_level` and `upper_fault_level` are fault thresholds:

- Values below `lower_fault_level` cause an error state.
- Values above `upper_fault_level` cause an error state.

An inactive analog limit may be represented as `NaN`.

### Reserved configuration field

Field number 19 is reserved.

The UI configuration message does not define a `debounced` field.

## Runtime request data points

Runtime request data points are defined in `UiReq`.

They are written using:

```text
PATCH /ui/<channel>
```

### Initialization values

| Data point | Field number | DI | Counter | Analog | Description |
|---|---:|:---:|:---:|:---:|---|
| `counter_init_val` | 1 | ✓ | ✓ | - | Sets the mode-specific counter value to the requested initial value |
| `init_on_time_s` | 2 | ✓ | - | - | Sets the accumulated Digital Input on-time value to the requested initial value in seconds |

The meaning of `counter_init_val` depends on the active mode:

| Mode | Effect |
|---|---|
| Digital Input | Initializes the secondary switching counter derived from rising edges of `digital_val` |
| Counter Input | Initializes the primary configured-edge hardware counter |
| Analog Input | Not valid |

`init_on_time_s` initializes the accumulated time for which the effective Digital Input value was `true`.

It is valid only for Digital Input mode because Counter Input and Analog Input modes do not provide `on_time_s`.

### Override control

| Data point | Field number | DI | Counter | Analog | Description |
|---|---:|:---:|:---:|:---:|---|
| `override_action` | 3 | G | G | G | Specifies the `OverrideAction` to apply to the UI channel |
| `override_duration_ms` | 4 | G | G | G | Specifies the requested override duration in milliseconds; a value of `0` means unlimited |
| `digital_override_val` | 5 | ✓ | - | - | Specifies the digital value used while a Digital Input override is active |
| `analog_override_val` | 6 | - | - | ✓ | Specifies the analog application value used while an Analog Input override is active |
| `counter_override_val` | 7 | - | ✓ | - | Specifies the counter value used while a Counter Input override is active |

The override value must match the active mode:

| Mode | Override value | Effect |
|---|---|---|
| Digital Input | `digital_override_val` | Temporarily replaces the processed Digital Input value |
| Counter Input | `counter_override_val` | Temporarily replaces the effective counter value |
| Analog Input | `analog_override_val` | Temporarily replaces the processed Analog Input value |

Initialization and override are different operations:

- `counter_init_val` changes the accumulated counter value.
- `init_on_time_s` changes the accumulated on-time value.
- An override temporarily replaces an effective process value.

## Runtime response data points

Runtime response data points are defined in `UiResp`.

They are returned using:

```text
GET /ui/<channel>
```

### General response data

| Data point | Field number | DI | Counter | Analog | Description |
|---|---:|:---:|:---:|:---:|---|
| `type` | 1 | G | G | G | Contains the active concrete `UiType` |
| `status` | 2 | G | G | G | Contains the current general point state represented by `PointStatus` |
| `error_code` | 3 | G | G | G | Contains the current I/O error represented by `IoErrorCode` |
| `override_remaining_ms` | 4 | G | G | G | Contains the remaining duration of the active override in milliseconds |
| `test_status` | 10 | G | G | G | Contains the current `TestStatus` of the I/O wiring test stored in the device |

The runtime status fields have different meanings:

| Data point | Meaning |
|---|---|
| `status` | General point state represented by `PointStatus` |
| `error_code` | Current I/O error represented by `IoErrorCode` |
| `test_status` | State of the I/O wiring test represented by `TestStatus` |
| `override_remaining_ms` | Remaining runtime of an active override in milliseconds |

### Mode-dependent response data

| Data point | Field number | DI | Counter | Analog | Description |
|---|---:|:---:|:---:|:---:|---|
| `digital_val` | 5 | ✓ | ✓ | - | Contains the effective logical Digital Input value after open-contact, inversion, and applicable override processing |
| `analog_val` | 6 | - | ✓ | ✓ | Contains the scaled counter representation or the fully processed Analog Input value, depending on the active mode |
| `counter_val` | 7 | ✓ | ✓ | - | Contains the secondary Digital Input switching counter or the primary Counter Input value |
| `on_time_s` | 8 | ✓ | - | - | Contains the accumulated time in seconds for which the effective Digital Input value was `true` |
| `sensor_val` | 9 | - | - | ✓ | Contains the sensor-level value before characteristic curve, two-point calibration, override, and analog limit processing |

The meaning of `digital_val` depends on the mode:

| Mode | Meaning |
|---|---|
| Digital Input | Primary effective digital process value |
| Counter Input | Underlying effective logical signal used for edge counting |

The meaning of `counter_val` depends on the mode:

| Mode | Meaning |
|---|---|
| Digital Input | Secondary switching counter that counts rising edges of `digital_val` |
| Counter Input | Primary counter value that counts rising or falling edges according to `inverted` |

The meaning of `analog_val` depends on the mode:

| Mode | Meaning |
|---|---|
| Counter Input | Counter value converted using `counter_scale` |
| Analog Input | Fully processed application value after characteristic curve, two-point calibration, override, and analog limit processing |

## Complete validity matrix: UI

### UI type resolution

| Resolved mode | Concrete Protobuf types | Description |
|---|---|---|
| Digital Input | `UI_TYPE_DI` | Digital Input with a digital value, switching counter, and accumulated on-time value |
| Counter Input | `UI_TYPE_COUNTER` | Counter Input with a digital source value, integer counter, and scaled counter representation |
| Analog Input | Every concrete `UI_TYPE_AI_*` | Analog Input with sensor-level and processed application values |

### UI configuration fields

| Data point | DI | Counter | Analog | Description |
|---|:---:|:---:|:---:|---|
| `enabled` | G | G | G | Indicates or configures whether the UI channel is active |
| `type` | G | G | G | Contains or selects the concrete `UiType` |
| `name` | G | G | G | User-defined channel name with a maximum length of 64 characters |
| `override_enabled` | G | G | G | Enables override control through the API or a connected display |
| `override_cancelable` | G | G | G | Allows an active override to be canceled through the API or a connected display |
| `inverted` | ✓ | ✓ | - | Inverts the Digital Input value or selects the counted edge in Counter Input mode |
| `open_contact` | ✓ | ✓ | - | Selects voltage-level or open-contact input wiring |
| `two_point_calibration` | - | - | ✓ | Optional linear correction defined by two points |
| `characteristic_curve` | - | - | ✓ | Optional conversion curve with two to 20 supporting points |
| `unit` | ✓ | ✓ | ✓ | Free-text engineering unit with a maximum length of 16 characters |
| `counter_scale` | - | ✓ | - | Scale factor for the Counter Input `analog_val` representation |
| `cov_increment` | ✓ | ✓ | ✓ | Minimum change in the process value that causes a change notification |
| `min_send_time_ms` | ✓ | ✓ | ✓ | Minimum change-notification interval in milliseconds |
| `max_send_time_ms` | ✓ | ✓ | ✓ | Maximum configured transmission interval in milliseconds |
| `minimum` | - | - | ✓ | Lower capping limit |
| `maximum` | - | - | ✓ | Upper capping limit |
| `lower_fault_level` | - | - | ✓ | Lower error threshold |
| `upper_fault_level` | - | - | ✓ | Upper error threshold |
| `false_text` | ✓ | ✓ | - | Display text for `false` with a maximum length of 16 characters |
| `true_text` | ✓ | ✓ | - | Display text for `true` with a maximum length of 16 characters |
| `configured` | G | G | G | Indicates or configures whether the channel is configured |
| `local_setup_enabled` | G | G | G | Enables local setup using a connected display |
| `test_status` | G | G | G | Contains or configures the `TestStatus` of the I/O wiring test |

Field number 19 is reserved. The UI configuration message does not define a `debounced` field.

### UI runtime request fields

| Data point | DI | Counter | Analog | Description |
|---|:---:|:---:|:---:|---|
| `counter_init_val` | ✓ | ✓ | - | Initializes the Digital Input switching counter or primary Counter Input value |
| `init_on_time_s` | ✓ | - | - | Initializes the accumulated Digital Input on-time value in seconds |
| `override_action` | G | G | G | `OverrideAction` applied to the channel |
| `override_duration_ms` | G | G | G | Requested override duration in milliseconds; `0` means unlimited |
| `digital_override_val` | ✓ | - | - | Digital value used during a Digital Input override |
| `analog_override_val` | - | - | ✓ | Analog application value used during an Analog Input override |
| `counter_override_val` | - | ✓ | - | Counter value used during a Counter Input override |

### UI runtime response fields

| Data point | DI | Counter | Analog | Description |
|---|:---:|:---:|:---:|---|
| `type` | G | G | G | Active concrete `UiType` |
| `status` | G | G | G | Current `PointStatus` |
| `error_code` | G | G | G | Current `IoErrorCode` |
| `override_remaining_ms` | G | G | G | Remaining override duration in milliseconds |
| `digital_val` | ✓ | ✓ | - | Primary Digital Input value or underlying logical Counter Input signal |
| `analog_val` | - | ✓ | ✓ | Scaled counter value or processed Analog Input value |
| `counter_val` | ✓ | ✓ | - | Digital Input switching counter or primary Counter Input value |
| `on_time_s` | ✓ | - | - | Accumulated Digital Input true-state time in seconds |
| `sensor_val` | - | - | ✓ | Sensor-level value before application processing |
| `test_status` | G | G | G | Current I/O wiring test status |

## UI type enum

### `UiType`

Used in:

- `UiCfgReq.type`
- `UiCfgResp.type`
- `UiResp.type`

| Value | Name | Description |
|---:|---|---|
| 0 | `UI_TYPE_DONT_CHANGE` | Retain the currently configured concrete UI type |
| 1 | `UI_TYPE_DEFAULT` | Use the device-specific default UI type; `UI_TYPE_AI_VOLTAGE` is the preferred default |
| 2 | `UI_TYPE_AI_VOLTAGE` | Voltage input, 0-10 V |
| 3 | `UI_TYPE_AI_CURRENT_0_20` | Current input, 0-20 mA |
| 4 | `UI_TYPE_AI_RESISTOR_10K` | Resistance input, 0-10 kΩ |
| 5 | `UI_TYPE_AI_RESISTOR_180K` | Resistance input, 0-180 kΩ |
| 6 | `UI_TYPE_AI_TEMPERATURE_PT1000_DIN` | PT1000 temperature input using the DIN characteristic |
| 7 | `UI_TYPE_AI_TEMPERATURE_PT1000_SAMA` | PT1000 temperature input using the SAMA characteristic |
| 8 | `UI_TYPE_AI_TEMPERATURE_NI1000_DIN` | Ni1000 temperature input using the DIN characteristic |
| 9 | `UI_TYPE_AI_TEMPERATURE_NI1000_SAMA` | Ni1000 temperature input using the SAMA characteristic |
| 10 | `UI_TYPE_AI_TEMPERATURE_NI1000_LG` | Ni1000 temperature input using the Landis & Gyr characteristic |
| 11 | `UI_TYPE_DI` | Digital Input |
| 12 | `UI_TYPE_COUNTER` | Counter Input |
| 13 | `UI_TYPE_AI_TEMPERATURE_NTC_10K` | NTC 10 kΩ temperature input |
| 14 | `UI_TYPE_AI_TEMPERATURE_NTC_10K_PRE` | Predefined NTC 10 kΩ temperature input |
| 15 | `UI_TYPE_AI_TEMPERATURE_NTC_20K` | NTC 20 kΩ temperature input |
| 16 | `UI_TYPE_AI_CURRENT_4_20` | Current input, 4-20 mA |

## Shared enums and message types

For more information, see [`shared/README.md`](../shared/README.md).

- `PointStatus`
  - Used in: `UiResp.status`
- `IoErrorCode`
  - Used in: `UiResp.error_code`, `IoErrorCodeSetting.val`
- `OverrideAction`
  - Used in: `UiReq.override_action`
- `TestStatus`
  - Used in: `UiCfgReq.test_status`, `UiCfgResp.test_status`, `UiResp.test_status`
- `Setting`
  - Used by the corresponding `*Setting` messages

## JSON serialization notes

Depending on the Protobuf-to-JSON conversion settings, a response may include fields with default values even if those fields are not meaningful for the active operating mode.

Consumers must use the active concrete `type` together with the validity matrix to determine which mode-dependent fields are meaningful.

Fields that are not valid for the active mode must not be interpreted as active process values.

## Example runtime response

### Analog Input response: `GET /ui/<channel>`

```json
{
  "type": "UI_TYPE_AI_VOLTAGE",
  "status": "STATUS_FAILURE",
  "errorCode": "ERR_PERIPHERY",
  "overrideRemainingMs": 0,
  "analogVal": 0,
  "sensorVal": 0,
  "testStatus": "TEST_OPEN"
}
```

In Analog Input mode:

- `sensorVal` is the sensor-level value after hardware-specific type conversion and error handling.
- `analogVal` is the fully processed application value.
- Digital and counter-specific fields are not meaningful.

## Example configuration response

### Analog Input configuration: `GET /ui/<channel>/cfg`

```json
{
  "enabled": true,
  "type": "UI_TYPE_AI_VOLTAGE",
  "name": "Voltage input 1",
  "overrideEnabled": true,
  "overrideCancelable": true,
  "twoPointCalibration": null,
  "characteristicCurve": null,
  "unit": "V",
  "covIncrement": 0.1,
  "minSendTimeMs": 100,
  "maxSendTimeMs": 60000,
  "minimum": 0,
  "maximum": 10,
  "lowerFaultLevel": null,
  "upperFaultLevel": null,
  "configured": true,
  "localSetupEnabled": false,
  "testStatus": "TEST_OPEN"
}
```