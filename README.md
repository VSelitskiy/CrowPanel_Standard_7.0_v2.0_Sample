# CrowPanel Standard 7.0 v2.0 — EEZ-Open Starter

A cleaned-up, ready-to-compile PlatformIO starter project for the **Elecrow CrowPanel 7" HMI display** (ESP32-S3).

This is a stripped-down version of the `Starter_EEZ-Open` example from [MikeWarriner/CrowPanel7inch](https://github.com/MikeWarriner/CrowPanel7inch), with all unnecessary artifacts removed and build issues fixed for the current PlatformIO toolchain.

---

## Hardware

- **Board:** Elecrow CrowPanel 7.0" HMI (ESP32-S3, 4MB Flash, OPI PSRAM)
- **Display:** 800×480 RGB, capacitive touch (GT911)
- **UI Framework:** [LVGL 8.4.x](https://lvgl.io/) + [EEZ Studio](https://www.envox.eu/eez-studio/)

---

## Prerequisites

1. Install [Visual Studio Code](https://code.visualstudio.com/)
2. Install the [PlatformIO extension](https://platformio.org/install/ide?install=vscode)
3. Install the custom board definition for the CrowPanel 7"

### Installing the board definition

Copy `BoardDefinition/esp32-s3-devkitc-1-myboard.json` to your PlatformIO boards directory:

**Windows:**
```
%USERPROFILE%\.platformio\platforms\espressif32\boards\
```

**Linux / macOS:**
```
~/.platformio/platforms/espressif32/boards/
```

> **Note:** The board definition included in this repository already contains the fix for `memory_type` (`opi_qspi`). The original file from Elecrow has a typo (`qspi_opi`) which causes the PlatformIO build system to look for a non-existent SDK folder, breaking compilation entirely.

---

## Building

1. Open this folder in VS Code
2. PlatformIO will automatically install all dependencies
3. Click **Build** (✓ in the status bar) or run:
```
pio run
```

### Key build flags

The following flags in `platformio.ini` are required to build LVGL correctly under the Arduino framework without ESP-IDF's Kconfig system:

```ini
build_flags =
    -D LV_LVGL_H_INCLUDE_SIMPLE
    -I./include
```

---

## Flashing

Connect the board via USB, then:
```
pio run --target upload
```

Or use **Upload and Monitor** in the PlatformIO sidebar.

---

## Project structure

```
├── BoardDefinition/
│   └── esp32-s3-devkitc-1-myboard.json   # Fixed board definition
├── include/
│   ├── lv_conf.h                          # LVGL configuration
│   └── touch.h                            # Universal touch controller driver
├── src/
│   ├── main.cpp                           # Entry point
│   ├── lgfx/                              # LovyanGFX display driver setup
│   └── ui/                               # EEZ Studio generated UI code
├── Starter.eez-project                    # EEZ Studio project — open this to edit the UI
├── Starter.eez-project-ui-state          # EEZ Studio UI state (open panels, zoom, etc.)
└── platformio.ini
```

To edit the UI, open `Starter.eez-project` in [EEZ Studio](https://www.envox.eu/eez-studio/). The generated code is exported to `src/ui/` and compiled as part of the PlatformIO build. `Starter.eez-project-ui-state` stores the editor's workspace state (open tabs, zoom level, panel layout) and does not affect the build.

---

## Customized files

Two sets of files in this project are specifically tailored for the CrowPanel 7" hardware and are not generic — they should be reviewed before reusing in other projects.

### `include/` — LVGL and touch configuration

- **`lv_conf.h`** — LVGL configuration with memory allocation via ESP heap (`heap_caps_malloc`), Arduino `millis()` as the tick source, RGB565 color depth, and all standard widgets and Montserrat fonts enabled.
- **`touch.h`** — Universal touch driver header supporting GT911 (active), FT6X36, and XPT2046 controllers, selectable via compile-time defines. Configured for the CrowPanel's GT911 on I2C pins SDA=19, SCL=20.

See [`include/README.md`](include/README.md) for full details.

### `src/lgfx/` — LovyanGFX display driver

- **`lgfx.h`** — Declares the `LGFX` class (16-bit parallel RGB bus, 800×480 panel, static LVGL draw buffer) and the global `lcd` instance.
- **`lgfx.cpp`** — Defines the complete RGB pin mapping (16 data lines + HSYNC/VSYNC/DE/PCLK), display timing parameters, LVGL flush and touch callbacks, and the `setup()` initialization sequence including backlight control via LEDC PWM on GPIO 2.

See [`src/lgfx/README.md`](src/lgfx/README.md) for full details.

---

## Dependencies

| Library | Version |
|---|---|
| lvgl/lvgl | ^8.4.0 |
| tamctec/TAMC_GT911 | ^1.0.2 |
| lovyan03/LovyanGFX | ^1.2.19 |
| adafruit/Adafruit GFX Library | ^1.12.5 |

---

## Troubleshooting

**Build fails with missing SDK headers (`sdkconfig.h`, `soc/efuse_reg.h`, etc.)**

The root cause is a typo in the original Elecrow board definition file. Make sure `esp32-s3-devkitc-1-myboard.json` contains `"memory_type": "opi_qspi"` and not `"memory_type": "qspi_opi"`. The corrected file is included in the `BoardDefinition/` folder of this repository.

---

## Credits

Based on [Starter_EEZ-Open](https://github.com/MikeWarriner/CrowPanel7inch/tree/main/Starter_EEZ-Open) by [MikeWarriner](https://github.com/MikeWarriner).  
UI designed with [EEZ Studio](https://www.envox.eu/eez-studio/).

## License

MIT