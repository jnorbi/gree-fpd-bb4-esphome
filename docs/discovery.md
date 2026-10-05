# Modbus discovery

The repository includes the discovery configuration used for the original FPD-BB4 Modbus exploration:

`esphome/modbus-discovery.yaml`

The public file is based directly on the original `gree_rs485_full_probe_v2.yaml`.

For public sharing, only the following were changed:

- Wi-Fi/API/web credentials were replaced with `!secret` references;
- web/API protection was added;
- user-facing Hungarian text was translated to English.

The actual scan logic, scan ranges, delays and known-register polling logic were preserved.

## Safety

The discovery configuration is intentionally **read-only**.

It uses only these Modbus functions:

- FC01 - Read Coils
- FC02 - Read Discrete Inputs
- FC03 - Read Holding Registers
- FC04 - Read Input Registers

No Modbus write action is included.

## Hardware and bus settings used by the original probe

- Seeed XIAO ESP32-C3
- Seeed Studio XIAO RS485 breakout board
- TX: GPIO6
- RX: GPIO7
- RS485 flow control / DE-RE: GPIO4
- 9600 baud
- 8 data bits
- no parity
- 1 stop bit
- slave address: 1
- `send_wait_time: 250ms`
- `turnaround_time: 200ms`

These are **Modbus RS485 settings**. They are unrelated to the separate WMBTC02 UART interface, which was captured at 4800 8E1.

## Automatic scans in the original probe

The probe scans one address at a time.

### Holding registers

`SCAN HOLDING 0-37`

- FC03
- addresses H0-H37
- count 1 per request
- 650 ms delay between addresses

### Input registers

`SCAN INPUT REGISTERS 0-37`

- FC04
- addresses I0-I37
- count 1 per request
- 650 ms delay between addresses

### Coils

`SCAN COILS 0-103`

- FC01
- addresses C0-C103
- count 1 per request
- 650 ms delay between addresses

### Discrete inputs

`SCAN DISCRETE INPUTS 0-103`

- FC02
- addresses D0-D103
- count 1 per request
- 650 ms delay between addresses

The probe records successful responses, Modbus exceptions, timeouts, custom responses and requests that were not sent.

## Manual reads

The original probe also provides manual read controls:

- start address: 0-65535
- count: 1-16
- Read Holding - FC03
- Read Input Register - FC04
- Read Coil - FC01
- Read Discrete Input - FC02

This was used to re-check interesting addresses after the broad scans.

## Known-value polling included in the probe

When no scan is active, the probe periodically reads the already identified values:

- H2-H4: mode, fan and setpoint
- H10: power
- H26: room temperature

This polling is also read-only.

## Results observed during the original exploration

From the actual scan:

- H0-H30 responded.
- H31-H37 timed out.
- C0-C87 responded.
- C88-C103 timed out.

A successful response does **not** mean every address in the responding range has a known function.

Only addresses confirmed by state changes are documented as known mappings in [Register map](register-map.md).

## How mappings were identified

The useful mappings were identified by changing one physical setting at a time and comparing the register/coil values.

Examples:

- operating mode -> H2;
- fan speed -> H3;
- target temperature -> H4;
- power -> H10;
- room temperature -> H26;
- timer -> C21;
- vertical swing -> C30;
- sleep -> C31;
- X-FAN -> C33.

## FC02 and FC04

The original probe scanned both Discrete Inputs and Input Registers as well.

The current repository does not claim a confirmed functional map for those two address spaces.

## Logging and normal use

The discovery firmware is intended for reverse engineering and diagnostics.

For normal day-to-day fan-coil control, use one of the main climate configurations instead.
