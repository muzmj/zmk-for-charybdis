# SuperMini ASD Test Shield

A minimal three-key standalone keyboard test for SuperMini nRF52840.

| Key | GPIO |
|---|---|
| A | P0.02 |
| S | P0.29 |
| D | P0.09 |
| Common row | P0.31 |

Connect each switch between P0.31 and its assigned column pin. Do not connect switches directly to 3.3 V or GND.

The shield supports direct USB and Bluetooth HID host connections. It is not a split keyboard and does not use a dongle. The PC is the Bluetooth Central; this keyboard acts as the Bluetooth HID Peripheral.

The ILI9341 display and XPT2046 touch controller are not configured.
