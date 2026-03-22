# src/lgfx/

This folder contains the LovyanGFX display driver configured specifically for the Elecrow CrowPanel 7" (800×480 RGB parallel interface) and its LVGL integration.

---

## lgfx.h

Declares the `LGFX` class and a global `lcd` instance used throughout the project.

### Class structure

```cpp
class LGFX : public lgfx::LGFX_Device
```

Inherits from LovyanGFX's base device class. Uses the ESP32-S3 native RGB bus and panel drivers:

```cpp
#include <lgfx/v1/platforms/esp32s3/Panel_RGB.hpp>
#include <lgfx/v1/platforms/esp32s3/Bus_RGB.hpp>
```

### Private members

| Member | Type | Description |
|---|---|---|
| `_bus_instance` | `lgfx::Bus_RGB` | 16-bit parallel RGB bus |
| `_panel_instance` | `lgfx::Panel_RGB` | RGB panel driver |
| `screenWidth` / `screenHeight` | `uint32_t` | Populated at runtime from `this->width()` / `this->height()` |
| `disp_draw_buf` | `lv_color_t[800*480/15]` | Static LVGL render buffer (~25KB at 16bpp) |
| `draw_buf` | `lv_disp_draw_buf_t` | LVGL draw buffer descriptor |
| `disp_drv` | `lv_disp_drv_t` | LVGL display driver descriptor |

### Public methods

| Method | Description |
|---|---|
| `LGFX(void)` | Configures bus and panel hardware parameters |
| `setup()` | Initializes display, LVGL, touch, and backlight |

### Global instance

```cpp
extern LGFX lcd;
```

`lcd` is defined in `lgfx.cpp` and used directly in `main.cpp` and `touch.h`.

---

## lgfx.cpp

Implements the `LGFX` class. Contains hardware pin assignments, display timing, and LVGL driver callbacks.

### RGB bus pin mapping

The display uses a 16-bit parallel RGB interface connected to the ESP32-S3 LCD/CAM peripheral:

| Channel | Pins (GPIO) |
|---|---|
| Blue B0–B4 | 15, 7, 6, 5, 4 |
| Green G0–G5 | 9, 46, 3, 8, 16, 1 |
| Red R0–R4 | 14, 21, 47, 48, 45 |
| HSYNC | GPIO 39 |
| VSYNC | GPIO 40 |
| DE (Data Enable) | GPIO 41 |
| PCLK | GPIO 0 |

### Display timing

| Parameter | Value |
|---|---|
| Pixel clock | 15 MHz |
| HSYNC front porch | 40 |
| HSYNC pulse width | 48 |
| HSYNC back porch | 40 |
| VSYNC front porch | 1 |
| VSYNC pulse width | 31 |
| VSYNC back porch | 13 |
| PCLK active edge | Negative |

### Panel geometry

| Parameter | Value |
|---|---|
| Resolution | 800 × 480 |
| Offset X / Y | 0 / 0 |

### LVGL callbacks

**`my_disp_flush()`**

Flushes a dirty rectangle from the LVGL render buffer to the display using LovyanGFX DMA transfer:

```cpp
lcd.pushImageDMA(area->x1, area->y1, w, h, (lgfx::rgb565_t *)&color_p->full);
```

DMA is used regardless of `LV_COLOR_16_SWAP` — both branches of the `#if` call the same function since byte swap is not needed for the parallel RGB interface.

**`my_touchpad_read()`**

Reads touch state from `touch.h` and feeds it to LVGL's input device system. Logs touch coordinates to Serial. Includes a 15 ms delay to debounce reads.

### `LGFX::setup()`

Initialization sequence called once from `main.cpp`:

1. `this->begin()` — starts the LovyanGFX driver
2. Fills screen blue as a visual boot indicator
3. `lv_init()` — initializes LVGL
4. `touch_init()` — initializes the GT911 touch controller
5. Allocates LVGL draw buffer: `screenWidth * screenHeight / 15` pixels (~25KB)
6. Registers display driver (`my_disp_flush` callback)
7. Registers touch input driver (`my_touchpad_read` callback)
8. Configures backlight on GPIO 2 via LEDC PWM channel 1 at 300 Hz, 8-bit resolution, brightness 255

### Backlight

```cpp
#define TFT_BL 2
```

Backlight is controlled via LEDC PWM. Brightness (0–255) can be adjusted by changing the last argument of:

```cpp
ledcWrite(1, 255);
```