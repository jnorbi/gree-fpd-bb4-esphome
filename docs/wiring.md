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

Connect the RS485 breakout board's:

- A -> fan-coil BMS/RS485 A
- B -> fan-coil BMS/RS485 B

Use the wiring/service documentation for your exact fan-coil model to identify the unit-side BMS/RS485 terminals.

The FPD-51BB4/A-K and FPD-68BB4/A-K belong to the same wall-mounted FPD-BB4 family, but this repository intentionally does not hard-code a terminal-block screw number as a universal instruction.

Power the XIAO/RS485 board from a suitable USB or regulated supply appropriate for the board.

## Important: the Wi-Fi connector is not the Modbus connector

The factory WMBTC02 Wi-Fi module uses a separate 4-wire connector.

That connector carries power and a separate UART-style communication interface. It is **not** the two-wire RS485 Modbus connection used by the main configuration in this repository.

Do not connect the RS485 A/B lines to the WMBTC02 UART pins.

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
