# Solar MPPT FOCV Tracker

Fractional Open Circuit Voltage (FOCV) tracker for the [CN3722 solar MPPT charger](https://www.maltepoeggel.de/?site=solar-mppt-cn3722). An ATtiny412 and an MCP4151 digital potentiometer replace the charger's fixed MPPT trimmer and follow the panel instead. Original project: [maltepoeggel.de](https://www.maltepoeggel.de/?site=solar-mppt-focv).

![Board, top side](docs/images/board-top.png)

*Render, not a photo.*

## Status

| Part | State |
|---|---|
| PCB | rev1, EasyEDA Pro, DRC 0 errors. Not manufactured |
| Firmware | Builds and passes host tests, not run on this board |
| Validation | Nothing built or measured |

Open items: [release notes](release/rev1/README.md#not-verified).

## Features

- Replaces the precision potentiometer of a CN3722 charger module
- Measures the panel open-circuit voltage every 2 s and sets the charger to 80 % of it
- Runs from the panel input through a 5 V LDO
- Status LED, UPDI programming header

## Specifications

| Item | Value |
|---|---|
| Board | 22.73 × 13.92 mm, 2 layers, 1.6 mm, parts on both sides |
| Input | 7 V to 28 V, including panel VOC at low temperature |
| MCU | ATtiny412 at 8 MHz |
| Digital potentiometer | MCP4151, 100 kΩ, 257 steps |
| ADC range | 27.8 V full scale, 18 kΩ / 3.3 kΩ divider, 4.3 V reference |
| Mounting | Bottom side faces the charger module, right-angle header K2 past the bottom edge |
| K2 pins | Pin 1 wiper, pins 2 and 3 GND; viewed from the top, GND, GND, wiper from left to right |

![Board, overview](docs/images/board-overview.png)

![Board, bottom side](docs/images/board-bottom.png)

## Wiring

ATtiny412 pins, see `src/pins.h`:

| Pin | Signal | Connects to |
|---|---|---|
| PA0 | UPDI | Programmer |
| PA1 | `PIN_POT_DA` | MCP4151 SDI |
| PA2 | `PIN_POT_CS` | MCP4151 CS |
| PA3 | `PIN_POT_CLK` | MCP4151 SCK |
| PA6 | `PIN_LED` / UART TX | Status LED or debug TX, not both |
| PA7 | `ADC_INP_VOLT` | Panel divider 18 kΩ / 3.3 kΩ |

U2 must be the 5 V `MCP1804T-5002I/OT` (C625416): the 4.3 V ADC reference and the 8 MHz clock need 5 V VDD. Panel Voc, also at low temperature, must stay below 28 V (MCP1804 limit; the ADC saturates at about 27.75 V).

## Firmware

PlatformIO, from the repository root: `pio run` builds, `pio run --target upload` flashes over UPDI (programmer and port in `platformio.ini`), `python3 tests/test_regulation.py` runs the host tests.

Build options in `src/main.cpp`, both off by default:

- `ENABLE_VMPP_LOOKUP`: take the potentiometer position from `vmpp_table` (`src/vmpp.h`) instead of the iterative loop. Faster, less accurate between table entries.
- `ENABLE_DEBUG`: print `VOC;VMPP[;steps];` at 9600 baud on PA6 instead of driving the LED.

Calibration constants:

| Constant | File | Meaning |
|---|---|---|
| `ADC_MAX_MV` | `src/adc.h` | Input voltage at ADC full scale: 4.3 V × (18 k + 3.3 k) / 3.3 k ≈ 27.75 V. Recalculate if the divider or reference changes |
| `K_PERCENT` | `src/main.cpp` | VMPP / VOC, 80 % for crystalline silicon |
| `ADC_DELAY_MS` | `src/main.cpp` | Settling time after a potentiometer step. Check with a scope if the panel or cabling changes |
| `VOC_MIN_MV` | `src/main.cpp` | Below this, the input counts as dark and charging stays off. 7 V, the MCP1804 minimum input |
| `VMPP_DEADBAND_MV` | `src/main.cpp` | Band around VMPP where the loop holds still |
| `MEAS_DELAY_MS` | `src/main.cpp` | Measurement cycle period. An overrun starts the next cycle at once |

The loop stops inside the deadband, or on crossing the target it keeps the position with the smaller error. Each VOC measurement starts a new search.

## Ordering

Upload [`release/rev1/gerber-rev1.zip`](release/rev1) and send:

```text
5PCS, FR4, 2 LAYERS, 1.6MM, 1OZ, Purple Soldermask, White Legend, Lead-free HASL, TENTED VIAS, order number will be added
```

| Item | Reason |
|---|---|
| 1.6 mm, 1 oz | Standard two-layer stack-up; the stack-up is not stored in the Gerbers |
| Lead-free HASL | 0805, SOIC and SOT-23 parts do not need flat pads |
| Smallest features | 0.25 mm tracks, 0.15 mm clearance, 0.3 mm via drill, 0.6 mm via pad |
| Stencil | Needed for each side if the board is reflowed at home |

## Bill of materials

See [`release/rev1/BOM.md`](release/rev1/BOM.md).

## Repository

| Path | Content |
|---|---|
| [`release/rev1`](release/rev1) | Ordering package: Gerber, BOM, placement |
| [`docs/images`](docs/images) | Renders |
| [`src`](src) | Firmware |
| [`tests`](tests) | Host tests |

The EasyEDA Pro project is not stored here.
