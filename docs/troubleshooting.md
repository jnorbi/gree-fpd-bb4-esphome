# Troubleshooting

## No Modbus communication

Check the basics first:

- Fan-coil powered on.
- Correct BMS/RS485 A and B connection.
- Correct slave address. The tested address is `1`.
- 9600 baud, 8 data bits, no parity, 1 stop bit.
- ESPHome TX = GPIO6.
- ESPHome RX = GPIO7.
- RS485 flow control = GPIO4.

Use `esphome/modbus-discovery.yaml` to issue read-only probes and inspect the logs.

## A/B polarity

RS485 A/B naming is not perfectly consistent across all manufacturers and interface boards.

Follow the markings and service documentation for the specific hardware. If communication is completely absent, verify polarity rather than assuming the names on two different devices use the same convention.

Power down before rewiring.

## Device appears online but commands occasionally fail

The normal controller currently uses:

- `send_wait_time: 250ms`
- `turnaround_time: 30ms`
- `max_cmd_retries: 2`

These values are used by the deployed working configuration.

The discovery configuration intentionally uses a more conservative `turnaround_time: 200ms`.

If testing a different FPD-BB4 revision and communication is unstable, temporarily increasing turnaround time is a reasonable diagnostic step before changing register logic.

## Heat mode does not appear in Home Assistant

There are two main configuration variants:

- `gree-fpd-bb4-cooling-only.yaml` intentionally does not expose Heat.
- `gree-fpd-bb4-heating-enabled.yaml` exposes H2=4 as Heat.

Use the variant that matches the hydronic system connected to the fan coil.

## "Unsupported Physical Mode" is active

This diagnostic exists only in the cooling-only configuration.

It means the physical fan coil is powered on and reports H2=4 (Heat), while the Home Assistant climate entity intentionally does not expose Heat mode.

This can happen if the mode was changed outside Home Assistant, for example with another controller.

## Auto mode confusion

There are two different concepts:

- **Operating-mode Auto:** not exposed. No separate practical Modbus operating-mode Auto was identified.
- **Fan-speed Auto:** supported and mapped to H3=0.

Fan Auto should therefore remain available even though HVAC Auto is absent.

## Dry mode fan-speed policy

The published controller accepts Auto and Low in Dry mode.

Low was directly observed to remain stable in Dry mode. Auto was also observed during a Cool-to-Dry transition. The configuration does not claim that other fan speeds are impossible at the hardware/protocol level; it conservatively changes Medium/High/Turbo to Low before entering Dry.

## X-FAN mode handling

C33 was directly observed to change with the X-FAN control.

The published configurations preserve the user's X-FAN preference across mode changes and only issue X-FAN ON writes in Cool/Dry. That Cool/Dry restriction is a conservative controller policy; it should not be read as a complete characterization of every mode combination supported by the fan-coil firmware.

## Turbo policy

H3=7 was directly observed when Turbo was selected.

The published controller exposes Turbo only in Cool mode. This is a conservative control policy, not a claim that the underlying firmware has been exhaustively tested for Turbo in every operating mode.

## Sleep in Heat

C31 was directly observed as the Sleep coil.

The published heating-enabled configuration only sends Sleep commands in Cool/Dry because Heat-mode Sleep behavior has not been field-tested. This is a controller limitation, not a proven fan-coil hardware limitation.

## Swing in Heat

Heat-mode swing behavior has not yet been field-tested by the repository author.

The heating-enabled configuration therefore does not send swing changes while the unit is physically in Heat mode.

## Display / light control is missing

Indoor-unit display/light control was not identified in the discovered RS485 Modbus map.

The factory WMBTC02/Gree+ path can control display/light, but a specific UART display/light command has not yet been decoded. See [UART reverse engineering](uart-reverse-engineering.md).

## External remote-control changes take a moment to appear

The main Modbus controller polls every 5 seconds.

Changes made outside Home Assistant can therefore take several seconds to be reflected by the climate entity.

## Why does the configuration use multiple-register writes for single values?

The deployed working configuration currently uses `use_write_multiple: true`.

An earlier independent reference implementation used single-register writes for the same H2/H3/H4/H10 values.

The current repository keeps the deployed write behavior unchanged because it works reliably. It can be revisited later if there is a demonstrated compatibility reason.
