# SuperMini GPIO Test Shield

This shield is for testing three physical switches on a SuperMini nRF52840 board using the existing Charybdis matrix pin assignments.

## Pin assignment

| Matrix signal | nRF GPIO | Connect |
| --- | --- | --- |
| Row 1 | P0.31 | One side of each switch |
| Column 1 | P0.02 | Other side of switch 1; sends `A` |
| Column 2 | P0.29 | Other side of switch 2; sends `S` |
| Column 3 | P0.09 | Other side of switch 3; sends `D` |

Each switch connects between the row and its corresponding column. Do not connect these switch contacts directly to 3.3 V or GND.

## Build

The root `build.yaml` includes this target as `nice_nano_v2` with shield `supermini_test`. Use the generated `supermini_test-nice_nano_v2-zmk.uf2` artifact if that is the exact artifact name provided by the build workflow.

## Important

- This test firmware replaces the currently installed keyboard firmware when flashed. Keep the existing keyboard firmware/build artifact available if you may want to restore it.
- This is a standalone test shield, not the Charybdis left peripheral configuration. It is intended to test USB keyboard input from the board itself.
- The build target is the repository's existing `nice_nano_v2` target. The SuperMini's UF2 bootloader identifies as `nice!nano`, but this alone does not guarantee every board-level detail is identical. Test with only USB and the three switches connected first.
- The ILI9341 display and XPT2046 touch controller are not configured by this shield yet.
