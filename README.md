# ZMK Charybdis - Keyboard Only

This branch is the **keyboard-only** variant of the Charybdis configuration.

## Hardware

- nice!nano v2 left/right keyboard
- PMW3610 trackball on the right half

## Firmware basis

The keyboard hardware definition and original PMW3610 configuration remain based on the Vzhao-L source. The keymap and common ZMK settings are synchronized with the cleaned `withDongle_v2` branch where they are applicable.

## Trackball

The original Vzhao-L PMW3610 driver is used:

- CPI: 400
- Snipe CPI: 200
- 125 Hz software polling
- Smart algorithm enabled
- Scroll tick: 70
- X inversion enabled
- 90-degree orientation
