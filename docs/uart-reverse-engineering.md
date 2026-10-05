# WMBTC02 UART reverse-engineering notes

This document describes a **separate interface** from the RS485 Modbus integration used by the main ESPHome configurations.

Everything below is limited to observations made on the tested fan-coil/WMBTC02 setup. It should not be assumed to apply unchanged to every Gree harness or hardware revision.

## Why this was investigated

The factory WMBTC02 Wi-Fi module communicates with the fan coil through a separate 4-wire serial connection.

The indoor-unit display/light function was not found in the confirmed RS485 Modbus map. The display/light can be controlled through the factory WMBTC02/Gree+ path, but a specific UART command/frame for the display/light function has **not** yet been identified.

## Measured 4-wire interface

On the tested harness, the fan-coil-side wire colors and measurements were:

| Wire | Observed function / level |
|---|---|
| Red | +5 V supply |
| Brown | GND |
| Yellow | Fan coil -> Wi-Fi module data; about 3.3 V idle with the Wi-Fi module disconnected |
| White | Wi-Fi module -> fan coil data; about 0.1-0.14 V measured with the Wi-Fi module disconnected |

With the WMBTC02 connected, communication activity was observed on both data wires.

These colors are observations from the tested harness, not a universal Gree pinout. Verify your own unit before connecting test equipment.

## UART settings actually used

Successful captures were made in HTerm using:

- 4800 baud
- 8 data bits
- Even parity
- 1 stop bit
- Flow control: off
- Display mode: HEX

In short: **4800 8E1**.

This is **not** the same as the RS485 Modbus interface used by the main controller, which runs at **9600 8N1**.

## Observed traffic

Both communication directions produced frames starting with:

`7E 7E`

Examples copied from the actual capture included:

```text
7E7E1C01000002020001F00000000000000000061100000000000000000029
7E7E1C01000002020001FA0000000000061100000000000000000033
7E7E1C01000101020001F00000000000000000041100000000000000000027
```

Additional `7E 7E 23 31 ...` status frames were also observed during testing.

These examples document observed traffic only. They do not imply that every byte in the protocol has been decoded.

## Capture directions

The directions identified during testing were:

- Yellow: fan coil -> WMBTC02
- White: WMBTC02 -> fan coil

A single USB-UART adapter can capture one direction at a time by connecting only its RX and GND.

To capture both directions simultaneously while keeping the transmitters electrically separate, two receive channels are useful: one RX channel for each data direction.

This is different from implementing an active bidirectional controller, where one TX and one RX signal are used.

## Logic-level caution

The ESP32-C3 GPIOs use 3.3 V logic.

The tested connector also carries a +5 V supply wire, so the supply voltage must not be confused with the data-line logic level. Measure the actual data lines and use appropriate level shifting before driving the interface from an ESP32.

## What is confirmed and what is not

Confirmed from the tests:

- the WMBTC02 uses a separate 4-wire serial interface;
- successful serial capture settings are 4800 8E1;
- both directions use `7E 7E`-prefixed frames;
- Yellow is fan coil -> Wi-Fi module;
- White is Wi-Fi module -> fan coil;
- the factory Wi-Fi path can control functions including display/light.

Not yet confirmed:

- a decoded display/light UART command;
- a complete byte-level protocol specification;
- compatibility of the observed wire colors/pin order with every FPD-BB4 revision.

## Scope of this repository

The supported production controller path in this repository is RS485 Modbus.

These UART notes are included only to document the separate WMBTC02 interface and to make continued reverse engineering reproducible without inventing protocol details that have not been verified.
