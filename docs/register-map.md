# Gree FPD-BB4 Modbus register map

This document contains only register mappings observed or confirmed during testing of Gree FPD-BB4 fan-coil units.

## Bus settings

- Modbus RTU over RS485
- 9600 baud
- 8 data bits
- No parity
- 1 stop bit
- Tested slave address: `1`

## Confirmed holding registers

| Address | Meaning | Observed values / range | Notes |
|---:|---|---|---|
| H2 | Operating mode | `1` = Cool, `2` = Dry, `3` = Fan, `4` = Heat | No separate Modbus Auto operating mode was found. |
| H3 | Fan speed | `0` = Auto, `1` = Low, `2` = Medium, `3` = High, `7` = Turbo | Fan-speed Auto is real and is unrelated to operating-mode Auto. |
| H4 | Temperature setpoint | `16..30` °C | Used by Cool/Dry and by the experimental Heat configuration. |
| H10 | Power | `85` / `0x0055` = Off, `170` / `0x00AA` = On | Confirmed by repeated state changes. |
| H26 | Room / return-air temperature | Temperature in °C | Exposed as a signed word in the current ESPHome configuration. |

### Example observed states

During reverse engineering, one observed Cooling / High / 22 °C state produced:

- H2 = `1`
- H3 = `3`
- H4 = `22`
- H10 = `170`
- H26 = `24`

A Dry / Low / 24 °C state produced:

- H2 = `2`
- H3 = `1`
- H4 = `24`

Power Off produced:

- H10 = `85`

These state changes were used to identify the registers rather than relying only on an external register map.

## Confirmed coils

| Address | Meaning | Notes |
|---:|---|---|
| C21 | Timer active | Read-only diagnostic in the main configuration. |
| C30 | Vertical swing | Boolean coil. |
| C31 | Sleep | Boolean coil. |
| C33 | X-FAN | Boolean coil. X-FAN is only enabled by the main configuration in Cool/Dry mode. |

## Address-range observations

The discovery work probed beyond the responding range to locate the apparent boundaries:

- Holding registers H0-H30 responded.
- H31-H37 timed out / did not respond successfully.
- Coils C0-C87 responded.
- C88-C103 timed out / did not respond successfully.

A responding address does **not** mean that every address in the range has a known or useful function. Only the mappings listed above are currently treated as confirmed.

## Heating

Holding register H2 value `4` is treated as **Heat**.

The remote control may present an Auto option, but no separate practical Modbus Auto operating mode was identified on the tested fan-coil setup. For this reason:

- `gree-fpd-bb4-cooling-only.yaml` detects H2=4 but does not expose Heat to Home Assistant.
- `gree-fpd-bb4-heating-enabled.yaml` exposes H2=4 as Home Assistant Heat mode.

The heating-enabled configuration is currently marked experimental because heating behavior has not yet been field-tested by the repository author. It conservatively enables Heat mode, setpoint control and normal fan-speed control. X-FAN, Turbo, Sleep and Heat-mode swing behavior are not assumed.

## Functions not found on Modbus

The indoor-unit display/light control was not identified in the discovered Modbus map. It was observed on the separate Wi-Fi-module UART protocol, which is outside the scope of the main RS485 integration.

## Write behavior

The current main ESPHome configuration uses `use_write_multiple: true` for its Modbus-controller entities. This is intentionally left unchanged because the deployed configuration is working reliably.

An earlier independently obtained reference configuration used single-register writes for H2/H3/H4/H10. Both approaches target the same confirmed register addresses and values.
