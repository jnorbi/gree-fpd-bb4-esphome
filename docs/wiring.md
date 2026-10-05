# Wiring

## Tested controller hardware

The working ESPHome setup uses:

- Seeed Studio XIAO ESP32-C3
- Seeed Studio XIAO RS485 expansion/breakout board

The ESPHome pin mapping used by this repository is:

| Function | ESP32-C3 pin |
|---|---:|
| RS485 UART TX | GPIO6 |
| RS485 UART RX | GPIO7 |
| RS485 DE/RE flow control | GPIO4 |

The bus settings are 9600 baud, 8 data bits, no parity, 1 stop bit.

## Fan-coil connection

The working controller uses the fan coil's dedicated BMS/RS485 differential pair.

Use the wiring/service documentation for your exact fan-coil model to identify the unit-side RS485 terminals. This repository intentionally does not publish a universal terminal-block screw number because the exact terminal labeling should be verified on the specific unit.

RS485 A/B naming is not perfectly consistent across vendors and interface boards, so follow the markings for your exact hardware rather than assuming that two devices use the same A/B convention.

Power the XIAO/RS485 board from a suitable USB or regulated supply appropriate for the board.

## Important: the Wi-Fi connector is not the Modbus connector

The factory WMBTC02 Wi-Fi module uses a separate 4-wire connector.

On the tested harness, the measured wires were Red = +5 V, Brown = GND, Yellow = fan coil -> Wi-Fi data and White = Wi-Fi -> fan coil data. Successful serial capture used 4800 baud, 8E1.

This is **not** the two-wire RS485 Modbus connection used by the main configuration, which runs at 9600 8N1.

Do not connect the RS485 A/B lines to the WMBTC02 UART data wires.

See [UART reverse engineering](uart-reverse-engineering.md) for notes about that separate interface.

## Modbus address

The tested Modbus slave address is:

`1`

The main ESPHome configurations therefore use:

```yaml
modbus_controller:
  - id: gree_controller
    address: 1
```

If your unit uses another Modbus address, update the configuration accordingly.

The discovery configuration exposes the slave address as a UI control, so it can be changed without editing the YAML.

## Before powering up

Check all of the following:

1. The XIAO is powered correctly.
2. RS485 A and B are connected only to the fan-coil's BMS/RS485 interface.
3. GPIO6 is used as TX.
4. GPIO7 is used as RX.
5. GPIO4 is used for RS485 flow control.
6. The fan-coil is powered.
7. The configured Modbus address matches the unit.
8. The bus is configured for 9600 8N1.

If there is no communication, power down before changing wiring.
