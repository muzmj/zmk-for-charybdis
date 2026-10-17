# SuperMini nRF52840 ASD Test

Minimal standalone ZMK keyboard firmware for a SuperMini nRF52840 board recognized by its UF2 bootloader as nice!nano.

## Keys and wiring

| Key | GPIO |
|---|---|
| A | P0.02 |
| S | P0.29 |
| D | P0.09 |
| Common row | P0.31 |

Each switch connects between the common row (P0.31) and its own column pin. Do not connect switch contacts directly to 3.3 V or GND.

## Direct PC connection

This is a single, standalone keyboard. Connect it to a PC using USB, or pair it directly with the PC over Bluetooth; no separate dongle or split-keyboard firmware is required. In Bluetooth HID, the PC is the Bluetooth Central and the keyboard is the Peripheral. ZMK's split Central role is not needed for this single-board setup.

## Build

GitHub Actions builds the `supermini_test` target. Download the artifact from the successful workflow run and flash the included UF2 through the board's UF2 bootloader.

Flashing replaces the firmware currently installed on the board. This branch intentionally contains only the ASD test configuration; the other branches are unchanged.

## Current scope

- Three-key A/S/D GPIO test
- USB and direct Bluetooth HID support
- No Charybdis left/right split, dongle, display, or trackball configuration
