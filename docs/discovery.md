# Modbus discovery

The repository includes a read-only discovery configuration:

`esphome/modbus-discovery.yaml`

It is a reconstructed and cleaned-up version of the workflow used while reverse engineering the Gree FPD-BB4 Modbus interface. The original historical scanner YAML was not preserved byte-for-byte, so this file should not be described as the original scanner.

## Safety

The discovery configuration is intentionally **read-only**.

It defines only these Modbus functions:

- FC01 - Read Coils
- FC02 - Read Discrete Inputs
- FC03 - Read Holding Registers
- FC04 - Read Input Registers

No Modbus write action is included.

## Hardware and bus settings

The discovery helper uses the same tested hardware mapping as the main configuration:

- Seeed XIAO ESP32-C3
- Seeed Studio XIAO RS485 breakout board
- TX: GPIO6
- RX: GPIO7
- RS485 flow control / DE-RE: GPIO4
- 9600 baud
- 8N1
- Tested slave address: 1
- `send_wait_time: 250ms`
- `turnaround_time: 200ms`

## How to use it

1. Copy `esphome/secrets.example.yaml` to `esphome/secrets.yaml` and fill in your own credentials.
2. Flash `esphome/modbus-discovery.yaml`.
3. Open the ESPHome device web interface or watch the ESPHome logs.
4. Set:
   - **Slave Address**
   - **Start Address**
   - **Register Count** for FC03/FC04
   - **Bit Count** for FC01/FC02
5. Press the relevant read button.
6. Inspect the `modbus_discovery` log entries.

The helper prints both decimal and hexadecimal values for register reads.

## Reproducing the FPD-BB4 exploration

The historical exploration attempted these ranges:

### Holding registers

Start with the known responding block:

- Start Address: `0`
- Register Count: `31`
- Read Holding Registers (FC03)

This covers H0-H30.

The next tested addresses H31-H37 did not respond successfully. When exploring unknown boundaries, probe one address or a small block at a time because a single unsupported address can cause a block request to fail.

### Coils

The known responding coil block was:

- Start Address: `0`
- Bit Count: `88`
- Read Coils (FC01)

This covers C0-C87.

C88-C103 did not respond successfully during the original exploration.

## Identifying a register

Do not assign meaning based only on a value that looks plausible.

Change exactly one physical setting at a time, read the same range again, and compare the result. This is how the known mappings were confirmed, for example:

- changing operating mode identified H2;
- changing fan speed identified H3;
- changing target temperature identified H4;
- toggling power identified H10;
- comparing room temperature identified H26.

The same change-one-variable-at-a-time method was used for the confirmed coils.

## FC02 and FC04

The public helper also exposes Discrete Input (FC02) and Input Register (FC04) reads so other users can continue investigating their own units.

The current repository does not claim a confirmed FPD-BB4 map for those two address spaces.

## Logging

The discovery YAML uses very verbose Modbus logging. It is intended for temporary reverse-engineering work, not as the normal day-to-day controller firmware.

For normal operation use one of the main configurations instead.
