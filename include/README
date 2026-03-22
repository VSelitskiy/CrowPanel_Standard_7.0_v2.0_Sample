# include/

This folder contains two configuration header files that must be present for the project to compile correctly.

---

## lv_conf.h

LVGL configuration file for the CrowPanel 7" display. Required by the build flags `-D LV_CONF_INCLUDE_SIMPLE` and `-D LV_KCONFIG_IGNORE` defined in `platformio.ini` — these tell LVGL to find `lv_conf.h` via the include path and skip the ESP-IDF Kconfig system entirely.

### Key settings

**Color**
| Setting | Value | Notes |
|---|---|---|
| `LV_COLOR_DEPTH` | 16 | RGB565 |
| `LV_COLOR_16_SWAP` | 0 | No byte swap (parallel RGB interface) |

**Memory**
| Setting | Value | Notes |
|---|---|---|
| `LV_MEM_CUSTOM` | 1 | Uses ESP heap allocator instead of LVGL's internal pool |
| `LV_MEM_CUSTOM_ALLOC` | `heap_caps_malloc(..., MALLOC_CAP_8BIT)` | Allocates from internal RAM |
| `LV_MEM_BUF_MAX_NUM` | 64 | Intermediate render buffers |

> The commented-out alternative uses `MALLOC_CAP_SPIRAM` to allocate from PSRAM. Switch to it if you run out of internal RAM in larger projects.

**Tick**
| Setting | Value | Notes |
|---|---|---|
| `LV_TICK_CUSTOM` | 1 | Uses Arduino `millis()` as tick source — no need to call `lv_tick_inc()` manually |

**Display**
| Setting | Value |
|---|---|
| `LV_DISP_DEF_REFR_PERIOD` | 30 ms |
| `LV_INDEV_DEF_READ_PERIOD` | 30 ms |
| `LV_DPI_DEF` | 130 |
| `LV_LAYER_SIMPLE_BUF_SIZE` | 24 KB |
| `LV_LAYER_SIMPLE_FALLBACK_BUF_SIZE` | 3 KB |

**Fonts**

All Montserrat sizes from 8 to 48 px are enabled. Default font is `lv_font_montserrat_14`.

**Widgets**

All standard and extra LVGL widgets are enabled, including Arc, Bar, Button, Canvas, Checkbox, Dropdown, Image, Label, Line, Roller, Slider, Switch, Textarea, Table, Calendar, Chart, Colorwheel, Keyboard, LED, List, Menu, Meter, Spinner, Tabview, Tileview, Window.

**Layouts**

Both Flex and Grid layouts are enabled.

**3rd party**

GIF and QR code decoders are enabled. PNG, BMP, JPG, FreeType, Rlottie and FFmpeg are disabled.

**Debugging**

`LV_USE_MEM_MONITOR 1` — memory usage overlay is shown in the bottom-left corner of the screen. Disable it in production by setting to `0`.

`LV_USE_LOG 0` — LVGL logging is disabled. Set to `1` and configure `LV_LOG_LEVEL` to enable debug output over Serial.

---

## touch.h

Universal touch driver header that supports three touch controllers via compile-time defines. Only one controller can be active at a time.

### Active configuration

The CrowPanel 7" uses a **GT911** capacitive touch controller. The following block is active (uncommented):

```c
#define TOUCH_GT911
#define TOUCH_GT911_SCL  20
#define TOUCH_GT911_SDA  19
#define TOUCH_GT911_INT  10
#define TOUCH_GT911_RST  11
#define TOUCH_GT911_ROTATION  ROTATION_NORMAL
#define TOUCH_MAP_X1  800
#define TOUCH_MAP_X2  0
#define TOUCH_MAP_Y1  480
#define TOUCH_MAP_Y2  0
```

### Supported controllers

| Controller | Interface | Notes |
|---|---|---|
| **GT911** *(active)* | I2C | Capacitive, used on CrowPanel 7" |
| FT6X36 | I2C | Capacitive, optional XY swap via `TOUCH_SWAP_XY` |
| XPT2046 | SPI | Resistive |

### API

The file exposes four functions used by the LVGL input driver:

| Function | Description |
|---|---|
| `touch_init()` | Initializes the touch controller (Wire/SPI bus + chip setup) |
| `touch_has_signal()` | Returns `true` if there is pending touch data to read |
| `touch_touched()` | Returns `true` if a touch is currently active; updates `touch_last_x` / `touch_last_y` |
| `touch_released()` | Returns `true` if the touch has been released |

Coordinates are mapped from raw sensor range to screen pixels using Arduino `map()` and stored in the global variables `touch_last_x` and `touch_last_y`.

### Switching to a different controller

Comment out the GT911 block and uncomment the block for your controller. Adjust the pin definitions and `TOUCH_MAP_*` values to match your hardware and screen resolution.