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

## Dry mode fan-speed limitation

The configuration treats only Auto and Low as valid fan speeds in Dry mode.

When switching to Dry from Medium, High or Turbo, the controller first changes the fan to Low.

## X-FAN turns off when switching to Fan mode

This behavior was observed on the unit.

The configuration remembers the user's X-FAN preference. If X-FAN was enabled, switching back to Cool or Dry restores it when needed.

X-FAN is not enabled in Heat or Fan mode by the published configurations.

## Turbo is unavailable

Turbo is intentionally exposed only in Cool mode.

Normal Auto/Low/Medium/High fan speeds remain separate from Turbo.

## Sleep does not work in Heat

The published heating-enabled configuration does not assume unverified Sleep behavior in Heat mode.

Sleep remains limited to Cool/Dry.

## Swing in Heat

Heat-mode swing behavior has not yet been field-tested by the repository author.

The heating-enabled configuration therefore does not send swing changes while the unit is physically in Heat mode.

## Display / light control is missing

Indoor-unit display/light control was not identified in the discovered RS485 Modbus map.

The function was observed on the separate WMBTC02 UART protocol instead. See [UART reverse engineering](uart-reverse-engineering.md).

## External remote-control changes take a moment to appear

The main Modbus controller polls every 5 seconds.

Changes made outside Home Assistant can therefore take several seconds to be reflected by the climate entity.

## Why does the configuration use multiple-register writes for single values?

The deployed working configuration currently uses `use_write_multiple: true`.

An earlier independent reference implementation used single-register writes for the same H2/H3/H4/H10 values.

The current repository keeps the deployed write behavior unchanged because it works reliably. It can be revisited later if there is a demonstrated compatibility reason.
