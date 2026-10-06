# ZMK Charybdis - Keyboard Only

This branch is the **keyboard-only** variant of the Charybdis configuration.

## Hardware

- nice!nano v2 left/right keyboard
- PMW3610 trackball on the right half
- No BLE central dongle
- No EC11 encoder
- No RGB / WS2812 LED

## Firmware basis

The keyboard hardware definition and original PMW3610 configuration remain based on the Vzhao-L source. The keymap and common ZMK settings are synchronized with the cleaned `withDongle_v2` branch where they are applicable.

The dongle-specific PMW3610-ALT and `zmk,input-split` configuration from `withDongle_v2` is **not** used here.

## Trackball

The original Vzhao-L PMW3610 driver is used:

- CPI: 400
- Snipe CPI: 200
- 125 Hz software polling
- Smart algorithm enabled
- Scroll tick: 70
- X inversion enabled
- 90-degree orientation

Cursor movement and scrolling behavior is tuned in the keymap to match the practical speed of `withDongle_v2` as closely as possible.

## Removed hardware features

The following unused features have been removed from the firmware:

- EC11 encoder Kconfig / Device Tree / keymap sensor bindings
- RGB / WS2812 Kconfig and Device Tree
- Encoder-related keymap behavior

The existing matrix GPIO definition, physical layout, Bluetooth settings, battery reporting, sleep settings, ZMK Studio settings, and user keymap are otherwise retained.
