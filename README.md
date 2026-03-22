# CrowPanel Standard 7.0 v2.0 — EEZ-Open Starter

A cleaned-up, ready-to-compile PlatformIO starter project for the **Elecrow CrowPanel 7" HMI display** (ESP32-S3).

This is a stripped-down version of the `Starter_EEZ-Open` example from [MikeWarriner/CrowPanel7inch](https://github.com/MikeWarriner/CrowPanel7inch), with all unnecessary artifacts removed and build issues fixed for the current PlatformIO toolchain.

---

## Hardware

- **Board:** Elecrow CrowPanel 7.0" HMI (ESP32-S3, 8MB Flash, OPI PSRAM)
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
    -D LV_CONF_INCLUDE_SIMPLE
    -D LV_KCONFIG_IGNORE
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
│   └── pins_arduino.h                     # Pin definitions
├── src/
│   ├── main.cpp                           # Entry point
│   ├── lgfx/                              # LovyanGFX display driver setup
│   └── ui/                               # EEZ Studio generated UI code
└── platformio.ini
```

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