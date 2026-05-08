# Adding a New Board to ESP-Claw

A comprehensive, vendor-neutral reference for contributing a board definition to the [espressif/esp-claw](https://github.com/espressif/esp-claw) repository. Written for engineers familiar with ESP-IDF who haven't worked with `esp_board_manager` before, and for LLMs assisting them.

This document is intended to be sufficient on its own: a fresh LLM session given this document plus the source materials it asks for in §2 should be able to produce a correct, building, verified-against-canonical-sources board definition. It supersedes earlier versions of the `espclaw_new_board.md` reference.

---

## 1. What you're building

A board definition tells ESP-Claw how to bring up the hardware on a specific development board so that the rest of the framework (LLM agent, Lua scripts, IM bots, display animations) can use it without knowing the specifics. ESP-Claw uses the [`esp_board_manager`](https://components.espressif.com/components/espressif/esp_board_manager) component to do this declaratively, with YAML files describing peripherals and devices.

A board definition consists of three to five files in a single directory. The directory drops into the ESP-Claw repository and the board becomes selectable via `idf.py bmgr -b <board_name>`.

---

## 2. The cardinal rule

**Never invent values.** If you don't have an authoritative source for a GPIO number, an I/O expander bit position, a display init byte, a timing parameter, or a flag, **ask the engineer for the source rather than generating plausible-looking digits.**

This rule exists because plausible-looking wrong values produce exactly the same observable symptom as completely missing values: the panel shows nothing, the touch panel doesn't respond, the audio codec is silent. Hours of debugging follow before anyone realizes the numbers were fabricated. The acceptable sources are listed in §3 below in priority order.

If a fresh LLM session finds itself about to generate a number it doesn't have a citation for, the correct action is to stop, name the missing source explicitly, and ask the engineer to provide it. The cost of one clarifying turn is low; the cost of a fabricated value is hours of bring-up debugging that ends with a rewrite.

---

## 3. Inputs to gather before writing any file

An LLM helping with bring-up should request all of the following from the engineer at the start of the session. Be explicit that proceeding without each item turns the work into guessing.

### 3.1 Board identity

- **Manufacturer**, short single-word form: `"Adafruit"`, `"Espressif"`, `"M5Stack"`, `"DFRobot"`, `"LilyGo"`, `"Seeed"`, `"Waveshare"` (not `"Adafruit Industries"`, etc.)
- **Marketing name** (e.g. "Qualia ESP32-S3 for RGB-666 Displays", "M5Stack CoreS3", "T-Display-S3")
- **Manufacturer SKU / product ID** (Adafruit 5800, M5Stack K128, etc.)
- **Chip**: `esp32`, `esp32s2`, `esp32s3`, `esp32c3`, `esp32c6`, `esp32h2`, `esp32p4`
- **Hardware revision** if multiple exist (`_revB`, `v1.5`, etc.) — some boards moved one or two GPIOs between revisions
- **Flash size** (4MB / 8MB / 16MB), **flash mode** (QIO/DIO/QSPI/OPI), **flash frequency** (80/120 MHz)
- **PSRAM** presence, **mode** (Quad/Octal), **frequency**
- **Console interface**: native USB CDC, or UART via USB-UART bridge (CP2102/CH340/FTDI). This determines whether `CONFIG_ESP_CONSOLE_USB_CDC=y` belongs in `sdkconfig.defaults.board`.

The conservative defaults are 80 MHz QIO flash and 80 MHz Octal PSRAM. Some boards' chips are rated for 120 MHz, but the safe choice for a first cut is 80 MHz; the manufacturer's existing firmware (whichever framework port the manufacturer maintains) tells you what's verified-stable.

### 3.2 Authoritative GPIO source — non-negotiable

Demand at least **one**, prefer **two or more for cross-checking**, of the following. Listed in priority order:

1. **Arduino BSP variant** — `pins_arduino.h` from `espressif/arduino-esp32/variants/<board_name>/`. This is most often the most reliable single source for raw GPIO numbers and I/O expander bit assignments because the manufacturer maintains it directly. Many manufacturers (Adafruit, M5Stack, LilyGo, Seeed, DFRobot, Waveshare) contribute their boards' variants here. **If this file exists, ask for it explicitly. Do not proceed without it unless none of the alternatives below are available either.**

2. **CircuitPython port** — `adafruit/circuitpython/ports/espressif/boards/<board_name>/pins.c` and `mpconfigboard.h`. Particularly valuable for I/O expander structures (e.g. `board.TFT_IO_EXPANDER`) and display init sequences (e.g. `board.TFT_INIT_SEQUENCE`), which CircuitPython exposes as auditable Python data. CircuitPython is most common for Adafruit boards but many other vendors have contributed entries.

3. **MicroPython port** — `micropython/ports/esp32/boards/<board_name>/`. The board directory contains `mpconfigboard.h` (GPIO macros, MCU type), `mpconfigboard.cmake` (verified flash/PSRAM/partition settings), and on some boards a `pins.csv` (tabular `Name,GPIO` mapping that's particularly easy for an LLM to parse). MicroPython is often the only port maintained for boards from smaller vendors or hobbyist communities.

4. **ESP-IDF native board support** — Espressif's own boards (ESP32-S3-EYE, ESP32-S3-LCD-EV-Board, ESP-BOX-3, ESP32-P4 Function EV) and many third-party boards have IDF-native definitions. Look in:
   - `examples/<category>/<example>/main/idf_board_support/` in the IDF tree
   - The IDF Component Registry: `https://components.espressif.com/` (search the manufacturer name)
   - YAMLs already in `esp_board_manager`: `https://github.com/espressif/esp-gmf/tree/main/packages/esp_board_manager/boards/`

   When an IDF-native source exists, it's the most authoritative for peripheral host/DMA selection (which SPI controller, which I2S port) and for `sdkconfig.defaults` content.

5. **Vendor schematic PDF or KiCad source** — definitive when ports disagree.

6. **Vendor's reference firmware** (Arduino sketch, vendor SDK fork) — useful for filling in details that ports don't expose.

**Do not** rely on marketing pinout diagrams alone. They are frequently stale, sometimes show pre-production layouts, and don't always match what shipped.

When multiple sources exist, **cross-check** between them. If they disagree, the schematic is the tiebreaker. Also confirm the board revision matches what the engineer has in hand.

### 3.3 Per-peripheral details (everything beyond raw GPIO)

For every non-trivial peripheral on the board, demand:

- **Part number of the silicon** — the actual driver IC, not the module name. For example: `ST7701` not `TL021WVC02CT-B1323`; `NV3052C` not `HD40015C40`; `Y17B` not `TL040HDS20`. Both should be recorded but they're not the same thing.
- **Datasheet** for the silicon
- **I2C address** in 7-bit form (clarify with the engineer if a source gives 8-bit shifted form)
- **Any board-specific configuration** — pull-up resistors on certain lines, voltage rails enabled by power management ICs, jumper-configurable defaults, solder-bridge alternatives

For displays specifically, also gather:

- **Display interface**: SPI, QSPI, MIPI DSI, RGB dot-clock, Intel-8080 parallel
- **Display resolution** in pixels
- **Pixel format at the host-to-panel wiring level** — RGB-565, RGB-666, RGB-888. **Count the data lines on the schematic; do not trust the marketing name.** A board sold as "RGB-666" may wire only 16 data lines (RGB-565) between MCU and panel.
- **Init byte sequence in canonical encoded form** (see §3.4 below)
- **Timing parameters**: pixel clock frequency in Hz, all six porches (hsync pulse_width / front / back, vsync pulse_width / front / back), and all polarity flags (`hsync_idle_low`, `vsync_idle_low`, `de_idle_high`, `pclk_active_neg`, `pclk_idle_high`)

For touch controllers: I2C address, interrupt GPIO routing (some boards route INT to the MCU; some route it through an I/O expander), reset GPIO routing, touch coordinate orientation flags (`mirror_x`, `mirror_y`, `swap_xy`).

For audio codecs: I2S configuration (master/slave, bit width, slot mode), I2C control bus, power-up sequencing requirements.

For power management chips: voltage rails enabled, currents, power-up sequencing.

### 3.4 Display init sequence — get the canonical encoded form

This is the single most error-prone piece of information in any board bring-up. Hand-transcription from plain-text init listings introduces errors at predictable spots — gamma tables, multi-byte parameter blocks, page-select sequences. These errors produce "no video" symptoms identical to "init never ran at all," and they are very hard to find by inspection.

The defense is to **work from the canonical encoded byte stream**, not the prose listing. Two acceptable forms:

**Form A — CircuitPython `bytes((...))` literal** (used with `dotclockframebuffer.ioexpander_send_init_sequence`):

```python
init_sequence = bytes((
    0xFF, 0x05, 0x77, 0x01, 0x00, 0x00, 0x10,   # cmd 0xFF, len 5, 5 data bytes
    0xC0, 0x02, 0x3B, 0x00,                      # cmd 0xC0, len 2, 2 data bytes
    0x11, 0x80, 0x64,                            # cmd 0x11, len 0x80 (= delay follows), delay 100ms
    0x29, 0x80, 0x32,                            # cmd 0x29, then 50ms delay
))
```

Encoding rules: each command is `<cmd_byte> <len_byte> <data_bytes...>`. If the high bit of `len_byte` is set (`0x80`), the low 7 bits give the parameter count, and the next byte after parameters is a delay in milliseconds.

**Form B — IDF C array** in the manufacturer's reference firmware. Often structured as `{cmd, {params...}, num_params, delay_ms}` or similar. Different shape; same information.

If the engineer hands you only a plain-text listing like `Wrt_Reg(0xFF, 0x30); Wrt_Reg(0xFF, 0x52); ...`, **request the encoded form as well** so you have something auditable. CircuitPython init sequences for many displays are visible directly on the manufacturer's product education pages or in the framework port's source.

When generating the C array for `setup_device.c`, parse the canonical encoded form programmatically rather than transcribing by eye.

### 3.5 Sanity checks the LLM should perform before writing files

- For every GPIO mentioned, confirm it's not in the SoC's reserved range. ESP32-S3 with Octal PSRAM reserves GPIO 26–37. Native USB on S3 uses GPIO 19–20.
- For every peripheral, confirm no two peripherals claim the same GPIO.
- For every I2C device, confirm the address doesn't collide with another device on the same bus.
- For every `type: custom` device, confirm whether the framework actually requires custom or whether a supported type would work.
- For every external dependency (`espressif/esp_lcd_ili9341`, etc.), confirm the package exists in the IDF Component Registry.

If any check fails, or any input from §3.1–3.4 is missing or ambiguous, ask the engineer rather than guessing.

---

## 4. Where the files go

Boards live under `application/edge_agent/boards/`, organized by manufacturer:

```
application/edge_agent/boards/
├── adafruit/
│   ├── metro_esp32s3/
│   └── your_board_name/
├── dfrobot/
│   └── dfrobot_k10/
├── espressif/
│   └── esp32_S3_DevKitC_1_breadboard/
├── lilygo/
│   └── lilygo_t_display_s3/
└── m5stack/
    └── ...
```

**Naming rules:**

- Manufacturer directory is lowercase, no spaces: `adafruit`, `dfrobot`, `lilygo`, `m5stack`, `espressif`, `seeed`, `waveshare`.
- Board directory is `[a-z0-9_]+` only — no hyphens, no uppercase, no spaces. Per `customize_board.md`: "Hyphens (`-`) or other special characters are not supported. If the name does not comply, this board will be unavailable."
- Modern convention: omit the manufacturer prefix from the board folder name when it would be redundant (e.g. `metro_esp32s3` not `adafruit_metro_esp32s3`). Some older boards (`dfrobot_k10`, `lilygo_t_display_s3`) still include the prefix for historical reasons; new boards should follow modern convention.
- The `board:` field in `board_info.yaml` must exactly match the board folder name.

---

## 5. The five files

| File | Required? | Purpose |
|------|-----------|---------|
| `board_info.yaml` | yes | Board metadata |
| `board_peripherals.yaml` | yes | Low-level bus configurations (I2C, SPI, RMT, I2S, GPIO, UART, LEDC) |
| `board_devices.yaml` | yes | High-level device configurations consuming the peripherals |
| `sdkconfig.defaults.board` | recommended | Board-specific Kconfig defaults |
| `setup_device.c` | only for `type: custom` devices | Init/deinit functions for non-supported device types |

Do **not** include in the board directory:

- `CMakeLists.txt` — the parent build system handles the board's source files
- `idf_component.yml` — dependencies belong inline in `board_devices.yaml`
- `setup_device.h` — board-private types live as `static` in `setup_device.c`
- `Kconfig.projbuild` — auto-generated by `idf.py bmgr` on first run

It is also customary to include a `MODULES.md` documenting the board's pin assignments, peripherals, and any quirks. This is for human readers and reviewers; it doesn't affect the build.

---

## 6. `board_info.yaml`

```yaml
board: your_board_name
chip: esp32s3
version: 1.0.0
description: "Manufacturer Product Name (16MB flash / 8MB OPI PSRAM)"
manufacturer: "Manufacturer"
```

- `board:` must exactly match the board folder name
- `chip:` is one of `esp32`, `esp32s2`, `esp32s3`, `esp32c3`, `esp32c6`, `esp32h2`, `esp32p4`
- `description:` is one terse sentence with `(NMB flash / NMB OPI/QIO PSRAM)` capacity hint at the end
- `manufacturer:` is the short single-word form

---

## 7. `board_peripherals.yaml`

Top-level `version: 1.0.0`, then a `peripherals:` list. Each entry:

```yaml
- name: <type_prefix>_<role>
  type: <i2c|spi|rmt|i2s|gpio|uart|ledc>
  role: <master|slave|tx|rx|none>
  config:
    <key>: <value>
```

**Critical rules:**

- YAML keys under `config:` must match the underlying ESP-IDF C struct field names exactly, with the same nesting. The generator translates YAML keys 1:1 into C struct initializers. Wrong field names → `field 'X' has no member` build errors.
- Peripheral names must start with their type prefix: `i2c_master`, `spi_display`, `rmt_tx`, `gpio_a0`, `uart_debug`, `i2s_audio_out`, `ledc_backlight`. Names are referenced by string from device entries — once chosen, keep them stable.

### 7.1 I2C

```yaml
- name: i2c_master
  type: i2c
  role: master
  config:
    port: 0
    pins:
      sda: 8
      scl: 18
```

Use the **nested `pins:` block** with short field names `sda`/`scl`. Older flat-style `sda_io_num`/`scl_io_num` is wrong for ESP-Claw.

### 7.2 SPI

```yaml
- name: spi_display
  type: spi
  role: master
  config:
    spi_bus_config:
      spi_port: SPI3_HOST       # SPI2_HOST or SPI3_HOST; SPI1 is reserved
      sclk_io_num: 39
      data0_io_num: 42          # MOSI on standard 4-wire SPI
      max_transfer_sz: 6400     # for displays: width * 2 bytes/px * 10 lines
```

- ESP-Claw convention is `SPI3_HOST` (both breadboard and metro use it)
- Omit MISO entirely if your bus is write-only (display)
- For general-purpose buses: `data1_io_num` (new driver) or `miso_io_num` (legacy); switch on field-mismatch error

### 7.3 RMT (NeoPixel)

```yaml
- name: rmt_tx
  type: rmt
  role: tx
  config:
    gpio_num: 46
    clk_src: RMT_CLK_SRC_DEFAULT
    resolution_hz: 10000000
    mem_block_symbols: 64
    trans_queue_depth: 4
    intr_priority: 1
    flags:
      invert_out: false
      with_dma: true
      io_loop_back: false
      io_od_mode: false
      allow_pd: false
      init_level: false
```

`clk_src` uses the **string form of the IDF enum** (`RMT_CLK_SRC_DEFAULT`), not the integer value.

### 7.4 GPIO, UART, I2S, LEDC

See the existing reference boards (especially `esp32_S3_DevKitC_1_breadboard`) for canonical examples of each.

---

## 8. `board_devices.yaml`

Top-level `version: 1.0.0`, then a `devices:` list. Each entry:

```yaml
- name: <device_name>
  chip: <chip_part_number>
  type: <device_type>
  sub_type: <subtype>             # only for some types
  version: default
  init_skip: false                # optional
  dependencies:
    espressif/<component>:
      version: "<semver>"
      public: true
  config:
    <fields>
  peripherals:
    - name: <peripheral_name>
      i2c_addr: [0x3F]            # required for I2C devices, 7-bit form
```

### 8.1 Supported device types

| `type` | Sub-types | Notes |
|--------|-----------|-------|
| `display_lcd` | `spi`, `dsi`, `parlio` | No `rgb` sub_type yet |
| `lcd_touch_i2c` | (none) | I2C touch panels |
| `audio_codec` | (none) | I2S audio codecs |
| `gpio_ctrl` | (none) | Simple GPIO control |
| `gpio_expander` | (none) | I2C GPIO expanders |
| `fs_fat` | `spi`, `sdmmc` | SD card or flash filesystem |
| `camera` | `dvp`, `mipi`, `usb` | Camera interfaces |

### 8.2 The `type: custom` escape hatch

Use when hardware doesn't fit a supported type. Two patterns:

**Pattern A — full custom device with init/deinit in `setup_device.c`:**

```yaml
- name: my_chip
  chip: chip_part_number
  type: custom
  version: default
  config:
    i2c_addr: 0x34
  peripherals:
    - name: i2c_master
```

You implement init/deinit functions in `setup_device.c` and register them with `CUSTOM_DEVICE_IMPLEMENT(my_chip, my_chip_init, my_chip_deinit)`.

**Pattern B — `init_skip: true` for application-managed devices:**

```yaml
- name: led_strip
  chip: ws2812
  type: custom
  version: default
  init_skip: true
  dependencies:
    espressif/led_strip:
      version: "^3.0"
      public: true
  config:
    max_leds: 1
  peripherals:
    - name: rmt_tx
```

Framework links the dependency but doesn't call init. Application instantiates on demand. **No `CUSTOM_DEVICE_IMPLEMENT` needed.**

### 8.3 I/O expanders — `type: gpio_expander`

```yaml
- name: gpio_expander
  chip: pca9554a
  type: gpio_expander
  version: default
  dependencies:
    espressif/esp_io_expander_tca9554: "*"
  config:
    max_pins: 8
    output_io_mask: [0, 1, 2, 4, 7]            # which bits are outputs
    output_io_level_mask: [0, 1, 0, 1, 0]      # initial levels for output bits, in order
    input_io_mask: [3, 5, 6]                   # which bits are inputs
  peripherals:
    - name: i2c_master
      i2c_addr: [0x3F]                         # 7-bit form
```

Notes:

- `output_io_level_mask` is **positional**: the Nth value gives the initial level for the Nth bit listed in `output_io_mask`. Not a bitmask.
- I2C address goes under the `peripherals:` reference, not at the top level.
- PCA9554A and TCA9554A are register-compatible (per `espressif/esp-bsp#335`), so the `esp_io_expander_tca9554` driver works for either.

### 8.4 Cross-device init order

If device A's init function needs device B, list B **first** in `board_devices.yaml`. The framework initializes devices in YAML order. From device A's init:

```c
some_handle_t *b_handle = NULL;
esp_board_device_get_handle("device_b_name", (void **)&b_handle);
```

---

## 9. `sdkconfig.defaults.board`

This file should contain **only board-hardware-specific Kconfig settings**. WiFi defaults, stack sizes, console settings, and other project-wide defaults belong in the project's top-level `sdkconfig.defaults`, not here.

Canonical minimum:

```
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_240=y
CONFIG_ESPTOOLPY_FLASHMODE_QIO=y
CONFIG_ESPTOOLPY_FLASHFREQ_80M=y
CONFIG_ESPTOOLPY_FLASHSIZE_16MB=y
CONFIG_PARTITION_TABLE_CUSTOM=y
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME="partitions_16MB.csv"
CONFIG_SPIRAM=y
CONFIG_SPIRAM_MODE_OCT=y
CONFIG_SPIRAM_SPEED_80M=y
```

Add as needed:

- `CONFIG_ESP_CONSOLE_USB_CDC=y` — boards with native USB only (no UART-bridge chip)
- `CONFIG_LCD_RGB_ISR_IRAM_SAFE=y` — RGB dot-clock displays (without it, flash operations during runtime corrupt the display)
- `CONFIG_ESP_VIDEO_ENABLE_USB_UVC_VIDEO_DEVICE=y` — USB cameras
- `CONFIG_ESP_LCD_TOUCH_CST816S_DISABLE_READ_ID=y` — CST8xx-family touch controllers (which only respond after a touch event)

ESP-Claw provides two partition tables in `application/edge_agent/`: `partitions_8MB.csv` (8MB flash) and `partitions_16MB.csv` (16MB, preferred).

---

## 10. `setup_device.c`

Required only if you have `type: custom` devices with `init_skip: false`. One file per board.

### 10.1 Standard structure

```c
/*
 * SPDX-FileCopyrightText: 2026 <your name or org>
 * SPDX-License-Identifier: Apache-2.0
 */

#include <string.h>
#include "esp_log.h"
#include "esp_check.h"
#include "esp_board_manager_includes.h"
#include "gen_board_device_custom.h"
/* chip-specific driver headers */

static const char *TAG = "BOARD_NAME_SETUP_DEVICE";

static int my_chip_init(void *config, int cfg_size, void **device_handle) {
    dev_custom_my_chip_config_t *cfg = (dev_custom_my_chip_config_t *)config;
    /* init logic */
    return ESP_OK;
}

static int my_chip_deinit(void *device_handle) {
    /* cleanup */
    return ESP_OK;
}

CUSTOM_DEVICE_IMPLEMENT(my_chip, my_chip_init, my_chip_deinit);
```

### 10.2 Required includes

```c
#include "esp_board_manager_includes.h"   /* MANDATORY — umbrella for board-mgr APIs */
#include "gen_board_device_custom.h"      /* MANDATORY — auto-generated struct defs */
```

For RGB-display custom devices that need to return `dev_display_lcd_handles_t`, you also need:

```c
#include "../../managed_components/espressif__esp_board_manager/devices/dev_display_lcd/dev_display_lcd.h"
```

The short form `#include "dev_display_lcd.h"` does not resolve on ESP32-S3 builds in this codebase — use the long relative path. (`lilygo_t_display_s3` uses this same path; mirror it.)

Do not include `esp_board_device.h` or `esp_board_periph.h` directly — use the umbrella.

### 10.3 The `CUSTOM_DEVICE_IMPLEMENT` macro

```c
CUSTOM_DEVICE_IMPLEMENT(<yaml_device_name>, <init_func>, <deinit_func>);
```

The first argument is the **YAML `name:` field** as a bare identifier (no quotes); the macro stringifies it.

Init/deinit signatures:

```c
int init_func(void *config, int cfg_size, void **device_handle);
int deinit_func(void *device_handle);
```

### 10.4 Auto-generated config struct field naming

The generator produces a struct named `dev_custom_<yaml_device_name>_config_t` with one field per YAML key under `config:`, plus auto-populated `chip` and `peripheral_name` strings. After the first `idf.py bmgr -b <board_name>`, open `components/gen_bmgr_codes/gen_board_device_custom.h` to see the exact field names; if your access pattern doesn't match, that's where to reconcile.

### 10.5 Getting peripheral and device handles

```c
/* I2C bus, SPI bus, etc. */
i2c_master_bus_handle_t bus = NULL;
esp_board_periph_get_handle(cfg->peripheral_name, (void **)&bus);

/* Other devices */
some_other_handle_t *other = NULL;
esp_board_device_get_handle("other_device_name", (void **)&other);
```

### 10.6 I/O expander factory function

If you have a `gpio_expander` device whose chip needs custom initialization (uncommon but happens), provide:

```c
esp_err_t io_expander_factory_entry_t(i2c_master_bus_handle_t i2c_handle,
                                      const uint16_t dev_addr,
                                      esp_io_expander_handle_t *handle_ret)
{
    esp_err_t ret = esp_io_expander_new_i2c_<chip>(i2c_handle, dev_addr, handle_ret);
    if (ret != ESP_OK) return ret;

    /* Set initial levels for any pins that need a specific state at boot. */
    esp_io_expander_set_level(*handle_ret, BIT(SOME_BIT), 1);
    return ESP_OK;
}
```

The framework finds this function by name. **Do not register via CUSTOM_DEVICE_IMPLEMENT.** No `static`.

### 10.7 Display factory function (supported `display_lcd` sub_types)

If your display uses a supported sub_type (`display_lcd:spi`, `display_lcd:dsi`, `display_lcd:parlio`) but with a chip the framework doesn't have a built-in factory for, provide:

```c
esp_err_t lcd_panel_factory_entry_t(esp_lcd_panel_io_handle_t io,
                                    const esp_lcd_panel_dev_config_t *panel_dev_config,
                                    esp_lcd_panel_handle_t *ret_panel)
{
    esp_lcd_panel_dev_config_t panel_dev_cfg = {0};
    memcpy(&panel_dev_cfg, panel_dev_config, sizeof(esp_lcd_panel_dev_config_t));
    return esp_lcd_new_panel_<your_chip>(io, &panel_dev_cfg, ret_panel);
}
```

**Do not register via CUSTOM_DEVICE_IMPLEMENT. No `static`.** The framework finds this by name and handles bus setup, panel reset, panel init, and turn-on. You only create the panel object.

### 10.8 RGB dot-clock displays — full custom

`esp_board_manager` v0.5.3 has no `display_lcd:rgb` sub_type. RGB dot-clock displays must use `type: custom` and return a `dev_display_lcd_handles_t *` from the init function:

```c
static int my_display_init(void *config, int cfg_size, void **device_handle) {
    dev_custom_display_lcd_config_t *cfg = (dev_custom_display_lcd_config_t *)config;
    dev_display_lcd_handles_t *h = calloc(1, sizeof(*h));
    ESP_RETURN_ON_FALSE(h, ESP_ERR_NO_MEM, TAG, "alloc failed");

    esp_lcd_rgb_panel_config_t panel_cfg = {
        .clk_src = LCD_CLK_SRC_DEFAULT,
        .timings = { /* from cfg */ },
        .data_gpio_nums = { /* see §11 — order matters */ },
        .data_width = 16,                  /* NOT num_data_lines */
        .de_gpio_num    = (int)cfg->de_gpio_num,
        .vsync_gpio_num = (int)cfg->vsync_gpio_num,
        .hsync_gpio_num = (int)cfg->hsync_gpio_num,
        .pclk_gpio_num  = (int)cfg->pclk_gpio_num,
        .disp_gpio_num  = -1,
        .bits_per_pixel = (uint8_t)cfg->bits_per_pixel,
        .flags = { .fb_in_psram = true },
    };

    esp_lcd_new_rgb_panel(&panel_cfg, &h->panel_handle);
    /* h->io_handle stays NULL — RGB has no separate IO controller */
    esp_lcd_panel_reset(h->panel_handle);
    esp_lcd_panel_init(h->panel_handle);
    esp_lcd_panel_disp_on_off(h->panel_handle, true);

    *device_handle = h;
    return ESP_OK;
}

CUSTOM_DEVICE_IMPLEMENT(display_lcd, my_display_init, my_display_deinit);
```

The device must be named `display_lcd` in YAML. ESP-Claw's `display_hal.c`, the display arbiter, and `lua_module_display` all expect a `dev_display_lcd_handles_t *` from `esp_board_device_get_handle("display_lcd", ...)`. Returning a board-private struct breaks the rest of the framework.

The IDF struct field is `data_width` (not `num_data_lines` — that name was deprecated in older IDF versions and removed; if you see it in old code, replace with `data_width`).

### 10.9 Bit-banged 3-wire SPI through an I/O expander (Qualia-style boards)

A common pattern: the board uses an I/O expander to drive 3-wire SPI to a display panel that needs an init sequence. The expander pins serve as CS, CLK, MOSI, RESET.

This pattern is used by Adafruit's Qualia line and several other RGB-666 carrier boards. The full implementation looks like this:

```c
#define PCA_BIT_CLK    /* per pins_arduino.h */
#define PCA_BIT_CS     /* per pins_arduino.h */
#define PCA_BIT_RESET  /* per pins_arduino.h */
#define PCA_BIT_MOSI   /* per pins_arduino.h */

/* Send one bit. CLK low, set MOSI, CLK high. */
static esp_err_t spi_send_bit(esp_io_expander_handle_t exp, int bit) {
    esp_io_expander_set_level(exp, BIT(PCA_BIT_CLK), 0);
    esp_io_expander_set_level(exp, BIT(PCA_BIT_MOSI), bit ? 1 : 0);
    esp_io_expander_set_level(exp, BIT(PCA_BIT_CLK), 1);
    return ESP_OK;
}

/* Send 9-bit word: D/CX bit then 8 data bits, MSB first.
 * D/CX = 0 for command, 1 for data. */
static esp_err_t spi_send_word9(esp_io_expander_handle_t exp,
                                bool is_data, uint8_t byte) {
    spi_send_bit(exp, is_data ? 1 : 0);
    for (int i = 7; i >= 0; i--) {
        spi_send_bit(exp, (byte >> i) & 1);
    }
    return ESP_OK;
}

/* Send one full register write: CS low, command word, parameter words, CS high.
 * Multi-parameter commands (common on ST7701 etc.) keep CS low across all bytes. */
static esp_err_t panel_write(esp_io_expander_handle_t exp,
                             uint8_t cmd, const uint8_t *params, size_t nparams) {
    esp_io_expander_set_level(exp, BIT(PCA_BIT_CS), 0);
    spi_send_word9(exp, false, cmd);
    for (size_t i = 0; i < nparams; i++) {
        spi_send_word9(exp, true, params[i]);
    }
    esp_io_expander_set_level(exp, BIT(PCA_BIT_CS), 1);
    esp_io_expander_set_level(exp, BIT(PCA_BIT_CLK), 0);   /* park CLK */
    return ESP_OK;
}
```

**The D/CX framing is what the panel datasheet specifies** (e.g. NV3052C §6.3). Each word is 9 bits: one D/CX bit followed by 8 data bits MSB-first. CS goes low at the start of a transaction and stays low until all parameters of that command have been sent.

**Speed notes:** I2C bus at 400kHz makes each `esp_io_expander_set_level()` take ~25–50µs, so each "SPI bit" takes ~50–100µs. A 159-command init sequence with ~3 bytes per command and 9 bits per byte traverses roughly 4,300 expander writes, taking 0.5–1 second. That's fine for boot but unsuitable for runtime traffic.

### 10.10 Init sequence encoding for `setup_device.c`

When transcribing a display init sequence into your C source, **work from the canonical encoded byte stream** (CircuitPython `bytes((...))` literal, or the manufacturer's reference firmware C array). Don't hand-transcribe from prose listings — you will introduce errors in gamma tables and multi-byte parameter blocks, and those errors look identical to "init never ran."

A clean format:

```c
#define CMD_OP   0x00
#define DELAY_OP 0xFF

static const uint8_t panel_init[] = {
    /* CMD_OP, len, cmd_byte, param_bytes... */
    CMD_OP, 5, 0xFF, 0x77, 0x01, 0x00, 0x00, 0x10,
    CMD_OP, 2, 0xC0, 0x3B, 0x00,
    /* DELAY_OP, milliseconds */
    DELAY_OP, 100,
    CMD_OP, 0, 0x29,                 /* DISPON, no params */
    DELAY_OP, 50,
};

static esp_err_t run_init(esp_io_expander_handle_t exp) {
    size_t i = 0;
    while (i < sizeof(panel_init)) {
        uint8_t op = panel_init[i++];
        if (op == DELAY_OP) {
            uint8_t ms = panel_init[i++];
            vTaskDelay(pdMS_TO_TICKS(ms));
            continue;
        }
        uint8_t nparams = panel_init[i++];
        uint8_t cmd = panel_init[i++];
        panel_write(exp, cmd, &panel_init[i], nparams);
        i += nparams;
    }
    return ESP_OK;
}
```

To produce the array from a CircuitPython `bytes((...))` literal: parse it programmatically (each command is `<cmd> <len> <data...>`; if `len & 0x80`, low 7 bits are the parameter count and a delay byte follows the parameters). Generate the C array from that parse rather than re-typing.

---

## 11. RGB data line ordering — the most expensive bug

For RGB dot-clock displays driven through `esp_lcd_new_rgb_panel()`, the `data_gpio_nums` array maps bit positions on the parallel bus to physical GPIOs. **Index 0 is the LSB of the bus, index N-1 is the MSB.**

The panel-side wiring determines which channel is the LSB. On most "RGB-666" 40-pin connectors (and on `RGB-565`-wired carriers like the Adafruit Qualia), the connector pin order is `DB0=B0` (blue LSB) up to `DB17=R5` (red MSB). For 16-line RGB-565 wiring, the order is then:

```c
.data_gpio_nums = {
    /* Blue LSB → MSB */
    cfg->data_gpio_b0, cfg->data_gpio_b1, cfg->data_gpio_b2,
    cfg->data_gpio_b3, cfg->data_gpio_b4,
    /* Green LSB → MSB */
    cfg->data_gpio_g0, cfg->data_gpio_g1, cfg->data_gpio_g2,
    cfg->data_gpio_g3, cfg->data_gpio_g4, cfg->data_gpio_g5,
    /* Red LSB → MSB */
    cfg->data_gpio_r0, cfg->data_gpio_r1, cfg->data_gpio_r2,
    cfg->data_gpio_r3, cfg->data_gpio_r4,
},
```

**Not** R first. Putting R first is one of the most common bring-up bugs and produces total-darkness or scrambled-noise symptoms identical to "panel never received pixel data."

**Verify against the panel datasheet's pin assignment table** before writing this array. The pin labels on the 40-pin connector tell you definitively which channel is the LSB.

Note that the YAML field names (`data_gpio_r0..r4`, `data_gpio_b0..b4`) are slot-based — they store the GPIO that drives **that channel's LSB**. Some panels (like RGB-565) don't have an `R0` or `B0` (only R1..R5 and B1..B5); in that case, populate the YAML's `data_gpio_r0` slot with the GPIO that drives R1 (the lowest red bit on the panel side). The slot naming is misleading but the values are what matter.

---

## 12. Pixel clock polarity — verify against the canonical source

`pclk_active_neg` controls whether pixel data is clocked on the rising or falling edge. Different panels need different settings. **Get this from the manufacturer's verified-working firmware**, not from inference.

If the source says `pclk_active_high=True` (CircuitPython convention), the YAML setting is `pclk_active_neg: false`. They mean the same thing. Don't invert this by accident.

If the source says `hsync_polarity=1` and `vsync_polarity=1` (Arduino-GFX convention), those typically mean active-high sync pulses, which corresponds to `hsync_idle_low: true` / `vsync_idle_low: true` in IDF terms — though check the specific panel's behavior, since polarity conventions differ between framework abstractions.

---

## 13. I/O expander direction & level masks — order-positional, not bitmasks

```yaml
config:
  max_pins: 8
  output_io_mask: [0, 1, 2, 4, 7]            # bits configured as outputs
  output_io_level_mask: [0, 1, 0, 1, 0]      # initial level for each output bit, IN ORDER
  input_io_mask: [3, 5, 6]                   # bits configured as inputs
```

The Nth value in `output_io_level_mask` gives the initial level for the Nth bit listed in `output_io_mask`. So in this example: bit 0 → 0, bit 1 → 1, bit 2 → 0, bit 4 → 1, bit 7 → 0. **It is not a bitmask** — `0b01010` would be wrong.

Backlight enable on many boards is wired through an I/O expander pin. **It must be configured as an output and driven HIGH** for the backlight to come on (the typical design uses a TPS61169 or similar driver whose enable pin needs an active-high signal). If the bit is left as input, behavior depends on whether the board has a hardware pull-up on that line:

- Hardware pull-up present → backlight comes on when the expander pin floats (input)
- No pull-up → backlight stays off until something drives it

Some Adafruit Qualia boards rely on the pull-up; some Arduino sketches drive HIGH explicitly. The safe approach is to always drive it HIGH explicitly: list the backlight bit in `output_io_mask` with a `1` in the corresponding `output_io_level_mask` slot, and as belt-and-suspenders, also call `esp_io_expander_set_level(handle, BIT(PCA_BIT_BACKLIGHT), 1)` in `io_expander_factory_entry_t()`.

---

## 14. Build and verification

### 14.1 First build

```bash
cd /path/to/esp-claw/application/edge_agent
idf.py bmgr -b your_board_name        # generates components/gen_bmgr_codes/
idf.py build
```

The first command produces:

- `components/gen_bmgr_codes/gen_board_periph_handles.c`
- `components/gen_bmgr_codes/gen_board_periph_config.c`
- `components/gen_bmgr_codes/gen_board_device_handles.c`
- `components/gen_bmgr_codes/gen_board_device_config.c`
- `components/gen_bmgr_codes/gen_board_info.c`
- `components/gen_bmgr_codes/gen_board_device_custom.h` ← types your `setup_device.c` casts to
- `components/gen_bmgr_codes/CMakeLists.txt`
- `components/gen_bmgr_codes/idf_component.yml`

**Run `idf.py bmgr -b <board_name>` after every YAML change.**

### 14.2 First flash

```bash
idf.py -p <port> flash monitor
```

Watch the boot log for `esp_board_manager` initializing each peripheral and device in YAML order. Each `setup_device.c` init function should log its progress (use `ESP_LOGI` liberally).

### 14.3 Common build errors

| Error | Cause |
|-------|-------|
| `gen_board_device_custom.h: no such file` | Forgot `idf.py bmgr -b <board_name>` |
| `field 'X' has no member` in `setup_device.c` | YAML key / C field name mismatch — open the generated header and reconcile |
| `dev_display_lcd.h: No such file or directory` | Use the long relative path from §10.2 |
| `'esp_lcd_rgb_panel_config_t' has no member named 'num_data_lines'` | IDF renamed it to `data_width` — change your code |

### 14.4 "No video" debugging ladder

When the board boots cleanly but the display shows nothing, work through these in order. Each rules out a different class of failure:

1. **Backlight visible at all?** Hold the panel up to a light source or shine a flashlight at an angle.
   - Yes → power and backlight enable path are fine; problem is in pixel data or panel config. Skip to step 4.
   - No → power, backlight enable, or backlight driver. Continue to step 2.

2. **Verify VCI is present** at the panel connector (multimeter on pin 5 of the FPC, or whatever pin the schematic shows for VCI). If VCI is missing, it's a board-level power problem, not software.

3. **Verify backlight enable signal.** If backlight is wired through an I/O expander pin, probe that pin with a multimeter. It should be HIGH (~3.3V) with the firmware running. If LOW or floating, see §13.

4. **Verify init sequence ran.** If the panel needs SPI init bytes, confirm the `setup_device.c` log output reaches the "init complete" message. If not, the init function is failing — check the I2C bus, the I/O expander handle, and the bit assignments in §13.

5. **Verify pixel data is reaching the panel.** Probe DCLK (PCLK) with a scope or logic analyzer at the panel connector. You should see the configured pixel clock frequency. If absent, `esp_lcd_new_rgb_panel()` failed silently or `esp_lcd_panel_disp_on_off(panel, true)` wasn't called.

6. **Verify polarity.** If pixel data is reaching the panel and the backlight is on but the screen is black or noise: invert `pclk_active_neg`. Many panels need the opposite of what the obvious choice suggests.

7. **Verify data line ordering.** See §11. Wrong ordering produces noise or solid black.

8. **Verify the init sequence is correct.** Diff your transcribed init bytes against the canonical encoded form (§3.4) byte-by-byte. A single wrong byte in a gamma table can produce no visible output.

The order matters: don't tweak polarity flags until you've confirmed the backlight is on, because you'll be guessing at what changed when.

### 14.5 Common runtime symptoms

| Symptom | Likely cause |
|---------|--------------|
| Display dark, no backlight | Backlight enable pin not driven HIGH; see §13 |
| Backlight on, screen solid black | Wrong `pclk_active_neg`, wrong init sequence, or wrong data line order |
| Backlight on, scrambled noise | Wrong data line order (§11), or wrong `bits_per_pixel` |
| Display shows wrong colors | RGB element order — try `LCD_RGB_ELEMENT_ORDER_BGR` vs `RGB`, or check byte endian |
| Display flickers | `pclk_hz` too high for this panel; drop until stable |
| Display works for a moment then corrupts | `CONFIG_LCD_RGB_ISR_IRAM_SAFE=y` not set |
| Boot splash doesn't appear | `app_claw_ui_start()` doesn't recognize your display's `panel_if`; check ESP-Claw integration |

---

## 15. Pre-PR checklist

- [ ] Folder placed at `application/edge_agent/boards/<manufacturer>/<board_name>/`
- [ ] Folder name is lowercase, alphanumeric, underscores only
- [ ] `board:` field in `board_info.yaml` matches the folder name exactly
- [ ] `manufacturer:` is the short single-word form
- [ ] `description:` is one terse sentence with `(NMB flash / NMB OPI/QIO PSRAM)` capacity hint
- [ ] `board_peripherals.yaml` and `board_devices.yaml` start with `version: 1.0.0`
- [ ] All peripheral names start with their type prefix (`i2c_*`, `spi_*`, etc.)
- [ ] All YAML config keys match the underlying ESP-IDF C struct field names
- [ ] No `CMakeLists.txt`, `idf_component.yml`, `setup_device.h`, or `Kconfig.projbuild` in the board directory
- [ ] `sdkconfig.defaults.board` contains only board-hardware-specific items (~10–15 lines)
- [ ] Partition table reference uses `partitions_8MB.csv` or `partitions_16MB.csv`
- [ ] If `setup_device.c` is present, every `type: custom` device with `init_skip: false` has a matching `CUSTOM_DEVICE_IMPLEMENT(name, init, deinit)`
- [ ] All GPIO assignments verified against an authoritative source named in §3.2
- [ ] Display init sequence (if any) verified against canonical encoded byte stream from §3.4, not hand-transcribed from prose
- [ ] RGB data line ordering verified per §11
- [ ] Pixel clock polarity verified against canonical source per §12
- [ ] I/O expander direction & level masks verified per §13
- [ ] `idf.py bmgr -b <board_name>` succeeds without errors
- [ ] `idf.py build` completes
- [ ] First flash boots, ESP-Claw initializes, captive portal accessible
- [ ] If display present: backlight on; image renders correctly
- [ ] Apache-2.0 license headers on any C source files contributed
- [ ] Author's `SPDX-FileCopyrightText` line in every C source file
- [ ] `MODULES.md` documenting pin assignments and board quirks

---

## 16. Reference boards in the repo

When in doubt, mirror an existing board:

| Board | Notes |
|-------|-------|
| `espressif/esp32_S3_DevKitC_1_breadboard` | Most complete reference: USB camera (custom), USB audio (custom), ST7789 SPI display, NeoPixel |
| `espressif/esp32_S3_DevKitC_1` | Bare DevKitC, no peripherals beyond defaults |
| `adafruit/metro_esp32s3` | Simple reference: ILI9341 SPI display, NeoPixel |
| `adafruit/qualia_esp32s3_rgb666` | RGB dot-clock with `type: custom`, no init sequence (Y17B-driven panel) |
| `adafruit/qualia_esp32s3_rgb666_round` | RGB dot-clock with bit-banged 3-wire SPI init (NV3052C) |
| `adafruit/qualia_esp32s3_rgb666_small_round` | RGB dot-clock + I2C touch (ST7701 + CST826) |
| `m5stack/m5stack_cores3` | Power-management chip example using `type: custom` (AXP2101) |
| `lilygo/lilygo_t_display_s3` | RGB dot-clock with the canonical `dev_display_lcd.h` include path |

---

## 17. How an LLM should structure a bring-up dialog

This document presumes an engineer is working with an LLM. Here's the recommended flow:

1. **LLM starts** by asking the engineer for everything in §3.1 (board identity).
2. **LLM asks for the highest-priority GPIO source from §3.2** that exists for this board. If the engineer can paste `pins_arduino.h`, that's typically sufficient. Otherwise, ask for `pins.c` + `mpconfigboard.h` from CircuitPython, or `pins.csv` + `mpconfigboard.cmake` from MicroPython.
3. **LLM enumerates every peripheral** beyond bare GPIO (§3.3) and asks for the silicon part numbers and datasheets.
4. **For each display**, LLM asks for the canonical encoded init sequence and the timing parameters (§3.4). If the engineer provides only a prose listing, LLM asks for the encoded form too — explain why (transcription errors are silent and devastating).
5. **LLM produces a side-by-side table** of every GPIO it plans to write, citing the source line for each. The engineer reviews before any file is generated.
6. **LLM generates files**, one at a time, showing the diff against any reference board it's mirroring.
7. **LLM runs the §3.5 sanity checks** before declaring done.
8. **After flash**, if the board doesn't work, LLM walks the engineer through §14.4 ("No video" debugging ladder) systematically rather than guessing.

The single highest-value habit: **when about to write a number that isn't directly traceable to a source the engineer provided, stop and ask.** This is the difference between a bring-up that takes 2 hours and one that takes 20.

---

## 18. Where to ask for help

- `esp_board_manager` issues: [github.com/espressif/esp-gmf](https://github.com/espressif/esp-gmf/issues)
- ESP-Claw issues: [github.com/espressif/esp-claw/issues](https://github.com/espressif/esp-claw/issues)
- Customize-board doc: `esp-gmf/packages/esp_board_manager/docs/how_to_customize_board.md`
- ESP-Claw docs: [esp-claw.com](https://esp-claw.com/en/)

---

## 19. What this document doesn't cover

- Adding new device types to `esp_board_manager` (e.g. RGB dot-clock as a first-class display sub_type) — that's an upstream contribution to `esp-gmf`
- Adding new peripheral types — same, upstream
- Modifying ESP-Claw application code (display arbiter, Lua modules, etc.) — application-level concerns
- Hardware bring-up (schematic verification, PCB testing) — assumes the hardware works and you're describing it for software
