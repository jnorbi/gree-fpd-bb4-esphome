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

The Modbus bus settings are 9600 baud, 8 data bits, no parity, 1 stop bit.

## FPD-BB4 RS485 terminals

For the Gree FPD-BB4 family, the BMS/RS485 connection is:

| Fan-coil terminal | RS485 |
|---:|---|
| 4 | A |
| 5 | B |

This terminal assignment comes from the FPD-BB4 documentation used during the project.

Gree support also confirmed that the internal terminal-compartment layout is shared across the FPD-BB4 family.

RS485 A/B naming can differ between third-party interface boards. On the fan-coil side, use the FPD-BB4 terminal assignment above; on the RS485 breakout side, verify the board's own A/B markings.

## Powering the ESP32 from the factory Wi-Fi connector

The factory WMBTC02 Wi-Fi module uses a separate **JST XA, 2.5 mm pitch, 4-pin** connector.

On the tested harness, the measured wire functions were:

| Wire | Observed function |
|---|---|
| Red | +5 V |
| Brown | GND |
| Yellow | Fan coil -> WMBTC02 serial data |
| White | WMBTC02 -> fan coil serial data |

A clean way to power the ESP32 installation is to use a mating **JST XA 4-pin cable** instead of cutting or soldering onto the fan-coil wiring.

### If the factory WMBTC02 is removed

A mating JST XA cable can be plugged directly into the factory Wi-Fi-module connector.

Use only:

- +5 V
- GND

for the ESP32/RS485 controller power supply.

The two serial-data wires are not required for the RS485 Modbus controller.

Pre-crimped JST XA 4-pin leads are available from suppliers such as AliExpress; the development installation used this type of cable.

### If the factory WMBTC02 is retained

Use a proper male-to-female JST XA pass-through / Y harness:

- pass all four WMBTC02 wires straight through;
- branch only +5 V and GND to the ESP32 controller;
- leave the two WMBTC02 serial-data lines connected only between the fan coil and the factory Wi-Fi module.

This keeps the factory connector system intact and avoids modifying the fan-coil PCB or harness.

## Important: WMBTC02 UART is not Modbus

The 4-wire WMBTC02 connector carries power and a separate serial interface.

Successful captures on the tested unit used:

- 4800 baud
- 8 data bits
- even parity
- 1 stop bit
- flow control off

In short: **4800 8E1**.

This is **not** the two-wire RS485 Modbus connection used by the main ESPHome controller, which runs at **9600 8N1**.

Do not connect the RS485 A/B lines to the WMBTC02 serial-data wires.

See [UART reverse engineering](uart-reverse-engineering.md) for the measured UART details.

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
2. Fan-coil terminal 4 is used for RS485 A.
3. Fan-coil terminal 5 is used for RS485 B.
4. GPIO6 is used as ESPHome Modbus TX.
5. GPIO7 is used as ESPHome Modbus RX.
6. GPIO4 is used for RS485 flow control.
7. The fan coil is powered.
8. The configured Modbus address matches the unit.
9. The Modbus bus is configured for 9600 8N1.
10. The WMBTC02 4-wire connector is not confused with the RS485 A/B connection.

If there is no communication, power down before changing wiring.
