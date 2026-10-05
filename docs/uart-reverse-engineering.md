# WMBTC02 UART reverse-engineering notes

This document describes a **separate interface** from the RS485 Modbus integration used by the main ESPHome configurations.

## Why this was investigated

The Gree WMBTC02 Wi-Fi module can control functions that were not found in the discovered RS485 Modbus map, including indoor-unit display/light control.

To investigate those functions, traffic between the fan coil and the factory Wi-Fi module was passively captured.

## Physical interface

The tested FPD fan coil uses a 4-wire connector for the factory WMBTC02 Wi-Fi module.

The outer wires were identified as power and ground. The two inner conductors carry bidirectional UART-style communication.

Wire colors and connector pin order should not be treated as universal across every harness or production revision. Measure and verify your own unit before connecting test equipment.

## UART settings observed

Traffic was successfully captured using:

- 9600 baud
- 8 data bits
- no parity
- 1 stop bit

Frames repeatedly started with:

`7E 7E ...`

Examples observed during testing included frames beginning with:

```text
7E 7E 1C 01 ...
7E 7E 1A 03 ...
7E 7E 23 31 ...
```

## Why passive sniffing needed two RX channels

There are two independent transmit directions:

1. fan coil -> Wi-Fi module
2. Wi-Fi module -> fan coil

For passive sniffing, each direction must be observed independently. Two receive channels make it possible to distinguish who transmitted each frame without electrically joining the two TX lines.

This is different from actively controlling the fan coil.

A normal active UART controller uses one TX and one RX pair, so a single bidirectional UART peripheral is enough.

## Logic-level caution

The ESP32-C3 GPIOs are 3.3 V logic.

Do not assume the fan-coil-side UART is directly safe for an ESP32 input/output. Measure the line levels and use appropriate level shifting when required.

The reverse-engineering setup therefore treated voltage-level compatibility separately from protocol decoding.

## What was learned

The UART work showed that the WMBTC02 interface carries functionality beyond the confirmed RS485 register map.

In particular:

- display/light control was visible through the UART path;
- the main RS485 integration did not identify a corresponding confirmed Modbus register/coil;
- the UART protocol is therefore a possible future extension, not part of the current production Modbus controller.

## Scope of this repository

The supported controller path is RS485 Modbus.

The UART notes are included so that other contributors can continue the reverse engineering without repeating the initial electrical/protocol discovery work.
