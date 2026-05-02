# Adding a New Board to ESP-Claw

A practical reference for contributing a board definition to the [espressif/esp-claw](https://github.com/espressif/esp-claw) repository. Written for engineers familiar with ESP-IDF who haven't worked with `esp_board_manager` before, and for LLMs assisting them.

---

## 1. What you're building

A board definition tells ESP-Claw how to bring up the hardware on a specific development board so that the rest of the framework (LLM agent, Lua scripts, IM bots, display animations) can use it without knowing the specifics. ESP-Claw uses the [`esp_board_manager`](https://components.espressif.com/components/espressif/esp_board_manager) component to do this declaratively, with YAML files describing peripherals and devices.

A board definition consists of three to five files in a single directory. The directory drops into the ESP-Claw repository and the board becomes selectable via `idf.py bmgr -b <board_name>`.

---

## 2. What you need before you start

Before writing any YAML, gather the following. **An LLM assisting with this work should explicitly check for each item and refuse to guess if any is missing** — guessing GPIO numbers or PCA9554A bit positions is the most common way these boards fail to build or fail to function correctly.

### 2.1 Mandatory inputs

**Authoritative GPIO source.** Use **all** of the following that exist for the board, in this priority order, and **cross-check between them**:

1. **ESP-IDF board support component**, if one exists. Espressif's own boards (ESP32-S3-EYE, ESP32-S3-LCD-EV-Board, ESP-BOX, etc.) and many third-party manufacturers (M5Stack, LilyGo, Waveshare, Seeed) publish IDF-native board support either:
   - As example code under `https://github.com/espressif/esp-idf/tree/master/examples/<category>/<example>/main/idf_board_support/`
   - As reusable components in the IDF Component Registry: `https://components.espressif.com/` (search the manufacturer name)
   - As YAML definitions already in `esp_board_manager` itself: `https://github.com/espressif/esp-gmf/tree/main/packages/esp_board_manager/boards/`

   When an IDF-native definition exists, it's the **most authoritative source** for ESP-Claw's purposes — it uses the same toolchain, it's version-aligned with the ESP-IDF release we're building against, and it often includes peripheral assignment details (SPI host selection, DMA channel preferences) that other ports don't expose. It also typically includes a verified `sdkconfig.defaults` you can crib from.

2. **A high-level framework port for the board**, if one exists. Different vendors maintain different ports:

   - **CircuitPython** (`https://github.com/adafruit/circuitpython/blob/main/ports/espressif/boards/<board_name>/pins.c` + `mpconfigboard.h`) — used by Adafruit, but also has community-contributed boards from other vendors. Best for boards Adafruit sells or supports, and tends to be the most complete framework port for chip-specific helper structures (I/O expander dictionaries, display init sequences).
   - **MicroPython** (`https://github.com/micropython/micropython/tree/master/ports/esp32/boards/<board_name>`) — broad community-driven coverage of ESP32 boards; often the only port maintained for boards from smaller vendors or hobbyist/maker communities. Provides `mpconfigboard.h` (GPIO macros, board name, MCU type), `mpconfigboard.cmake` (flash/PSRAM/partition settings), and sometimes a `pins.csv` (tabular pin-name-to-GPIO mapping that's especially easy for an LLM to parse).
   - **Arduino BSP** (`https://github.com/espressif/arduino-esp32/blob/master/variants/<board_name>/pins_arduino.h`) — most widely supported across vendors. Espressif maintains the upstream variants directory and accepts manufacturer contributions; Adafruit, DFRobot, M5Stack, LilyGo, Seeed, and Waveshare all publish variants there.
   - **Vendor's own SDK or BSP** — some manufacturers (M5Stack, LilyGo) maintain their own SDK distributions independent of Arduino-ESP32. Check the manufacturer's GitHub organization.

   These ports are typically the most complete source for **chip-specific details** that don't appear in IDF ports — I/O expander bit positions, display init byte sequences, register configuration tables. Even if you have an IDF source, check the framework port for these details.

   **Why these are valuable:** they're verified-working (the manufacturer ships software against them) and they often include canonical macro names that match the manufacturer's documentation and forum support. For example, Adafruit's Arduino variants define macros like `PCA_TFT_BACKLIGHT`, `PIN_NEOPIXEL`, `PIN_I2C_POWER` that match their published example sketches, making it easy to cross-reference user discussion.

3. **The official schematic** from the manufacturer (PDF or KiCad source). Definitive when ports disagree.

4. **The vendor's reference firmware source** (often a GitHub example repo).

**Cross-checking rule:** When multiple sources exist, generate your YAML based on whichever has the most **detail relevant to your specific need**:
- For raw GPIO numbers: ESP-IDF source (if present), then any framework port — they should all agree.
- For I/O expander bit positions and display init bytes: framework ports (CircuitPython, MicroPython, or Arduino), in roughly that order of detail.
- For peripheral host/DMA selection: ESP-IDF source if present; otherwise the manufacturer's reference firmware.
- For canonical macro names matching forum support: Arduino BSP (most widely cross-referenced).

If sources disagree on a GPIO number, check commit dates — boards sometimes have hardware revisions that update one source before the other. The schematic is the tiebreaker. Also note that some boards have `_revB` or similar variants — make sure all sources are for the revision you actually have.

**Do not use marketing-page pinout diagrams as your sole source.** They're frequently stale, sometimes show pre-production layouts, and don't always match what shipped.

**Chip identification for every non-trivial peripheral on the board.** Examples:
- Display panel chip (ST7789, ILI9341, GC9A01, TL040HDS20, HD40015C40, etc.)
- I/O expander chip (PCA9554A, AW9523, MCP23017, etc.)
- Touch controller (FT5x06, CST816S, GT911, etc.)
- Audio codec (ES8311, AW88298, ES7210, etc.)
- Power management chip (AXP192, AXP2101, etc.)
- IMU, environmental sensors, anything with an I2C/SPI address

For each, you need the part number, I2C address (if applicable), and any board-specific configuration (pull-up resistors, voltage rails, reset pulse requirements).

**Memory configuration.** Flash size, flash mode (QIO/DIO/QSPI/OPI), flash speed, PSRAM presence, PSRAM mode (QPI/OPI), PSRAM speed. The conservative defaults are 80 MHz QIO flash and 80 MHz Octal PSRAM. Some boards' chips are rated for 120 MHz, but the safe choice for a first cut is 80 MHz; the manufacturer's existing firmware (whichever framework port the manufacturer maintains) tells you what's verified-stable.

**Console interface.** Native USB CDC (no UART bridge chip on board) or UART via USB-UART bridge (CP2102, CH340, FTDI). This determines whether you need `CONFIG_ESP_CONSOLE_USB_CDC=y` in `sdkconfig.defaults.board`.

### 2.2 Display-specific inputs (if the board has a display)

**Display chip part number.** Not just "TFT" — the actual driver IC (ILI9341, ST7789, etc.) and the panel module (TL040HDS20, HD40015C40, etc.).

**Display interface.** SPI, QSPI, MIPI DSI, RGB dot-clock, Intel-8080 parallel. This determines whether you can use `type: display_lcd:spi` (easy, framework does most of the work) or need `type: custom` (full hand-rolled init).

**Display resolution.** Width × height in pixels, in landscape orientation by default.

**Pixel format.** RGB-565 (16-bit), RGB-666 (18-bit), RGB-888 (24-bit). For ESP32-S3 RGB displays, even when the panel accepts RGB-666, the chip-to-panel wiring is often only 16 lines (RGB-565). Check the schematic — count the data lines from MCU to panel.

**Timing parameters** (for non-SPI displays). All of:
- Pixel clock frequency (in Hz)
- Horizontal: pulse width, back porch, front porch
- Vertical: pulse width, back porch, front porch
- Signal polarity flags: `hsync_idle_low`, `vsync_idle_low`, `de_idle_high`, `pclk_active_neg`, `pclk_idle_high`

Source for these: the panel datasheet (preferred), the manufacturer's reference firmware (acceptable), or any framework port's panel driver if one exists. Manufacturers often publish display drivers as Python or C source in their respective port libraries — for example, Adafruit publishes RGB-666 panel timings in `Adafruit_CircuitPython_Qualia/adafruit_qualia/displays/*.py`. Look for similar driver files in whatever framework port the manufacturer maintains.

**Panel init sequence** (for displays needing one). Some panels work with no init bytes (TL040HDS20). Most need a sequence of register writes. Source: same as timings, plus the panel chip's datasheet command list.

**Backlight control.** Direct GPIO? PWM via LEDC? Driven through an I/O expander pin? Active-high or active-low? Any default current setting that's hardware-configurable (jumpers, solder bridges, resistor selection)?

### 2.3 Specialized-hardware inputs

If the board has any of the following, you need additional documentation:

**I/O expanders** (PCA9554A, AW9523, etc.):
- I2C address (often jumper-configurable — check default and any solder-bridge alternatives)
- Pin assignments for every signal: which expander bit drives backlight, which drives reset, which reads buttons, etc.
- Initial direction register value (which pins are inputs vs outputs at boot)
- Initial output register value (resting state of output pins)

For boards that document their I/O expander wiring publicly (Adafruit's CircuitPython port exposes `board.TFT_IO_EXPANDER` as a dict with the bit positions and init bytes; other vendors publish equivalent structures in their ports), use the documented values directly. Don't guess these — wrong direction-register bits can leave inputs floating or shorted to ground, and wrong bit-to-signal mappings will activate the wrong hardware.

**Power management chips.** Voltage rails enabled, currents, sequencing (some require specific power-up order). The cores3 board's `power_manager.c` is the reference for AXP2101.

**Touch controllers.** I2C address, interrupt GPIO, touch coordinate orientation (mirror_x, mirror_y, swap_xy depending on how the touch panel is mounted relative to the display).

**USB host hardware** (UAC, UVC). The breadboard's `setup_device.c` is the reference. Note the reference-counted shared USB host pattern — multiple USB devices share one host stack.

**Anything else not in the supported types list** (`display_lcd`, `lcd_touch_i2c`, `audio_codec`, `gpio_ctrl`, `gpio_expander`, `fs_fat`, `camera`). It needs `type: custom` and a full init/deinit pair in `setup_device.c`.

### 2.4 Sanity checks the LLM should perform

Before generating files, an LLM should explicitly verify:

- [ ] For every GPIO mentioned, confirm it's not in the SoC's reserved range (e.g., GPIO 26-37 on ESP32-S3 with Octal PSRAM, GPIO 19/20 if using native USB).
- [ ] For every peripheral, check no two peripherals claim the same GPIO.
- [ ] For every I2C device, confirm the address doesn't collide with another device on the same bus.
- [ ] For every `type: custom` device, confirm whether the framework actually requires custom or whether a supported type would work.
- [ ] For every reference to an external component (`espressif/esp_lcd_ili9341`, etc.), confirm the package exists in the IDF Component Registry.

If any check fails or any input from §2.1-2.3 is missing or ambiguous, **the LLM should ask the engineer rather than guessing**. The cost of one clarifying question is low; the cost of a fabricated GPIO that compiles but doesn't function is hours of bring-up debugging.

---

## 3. Working with an LLM on this task

This document is structured so that an engineer with the hardware information from §2 can hand it to an LLM along with the board's source materials (port-specific pin definitions, schematic, datasheet excerpts) and get correct files out. A few conventions make this work better:

**Provide the LLM with the source files directly.** If the engineer has framework-port files for the board (CircuitPython `pins.c`, MicroPython board config, Arduino `pins_arduino.h`, ESP-IDF Kconfig + headers — whichever exist), paste their full contents into the LLM context. The LLM can then cite specific lines as it generates, which is auditable. Don't summarize — the LLM should read the originals.

**Identify exactly which hardware variant the board has.** Many boards ship in multiple configurations under one product line — different display panels, different memory sizes, different sensor packages. The board file is typically the same across variants, but device entries (display chip, timings, init sequences) differ. Specify which variant before generating files. As one example, the Adafruit Qualia ESP32-S3 RGB-666 ships with several display options (TL040HDS20 4" square, HD40015C40 4" round, TL021WVC02 2.1" round, etc.); the same board file works for all of them but the display device's `chip:`, timings, and init sequence change per variant.

**Hand the LLM at least one reference board's complete file set.** Even with this document, an LLM benefits from seeing a full working example to mirror. The two cleanest references are the `esp32_S3_DevKitC_1_breadboard` board (under the `espressif` namespace) for a complex multi-device board with USB hosting and a SPI display, and the `metro_esp32s3` board (under the `adafruit` namespace) for a simpler board with one SPI display and a NeoPixel.

**Verify the generated GPIO assignments before flashing.** Have the LLM produce a side-by-side table of every GPIO it assigned vs the source it cites. The engineer reviews the table. This catches both LLM errors and source-document errors.

**Don't ask the LLM to invent timing parameters.** Display timings (`hsync_pulse_width`, etc.) come from the panel datasheet or the manufacturer's verified firmware. If neither is available for the panel, the panel cannot be supported until that information is obtained — there is no useful inference path.

---

## 4. Where the files go

ESP-Claw boards live under `application/edge_agent/boards/`, organized by manufacturer:

```
application/edge_agent/boards/
├── adafruit/
│   ├── metro_esp32s3/         <- existing
│   └── your_board_name/       <- yours
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

- The **manufacturer directory** is lowercase, no spaces (`adafruit`, `dfrobot`, `lilygo`, `m5stack`, `espressif`).
- The **board directory** is `[a-z0-9_]+` only — no hyphens, no uppercase, no spaces. Per `customize_board.md`: "Hyphens (-) or other special characters are not supported. If the name does not comply, this board will be unavailable."
- Modern Adafruit-namespace convention: **omit the manufacturer prefix from the board folder name** (e.g., `metro_esp32s3` not `adafruit_metro_esp32s3`). Some older boards (`dfrobot_k10`, `lilygo_t_display_s3`) still include the prefix.
- The `board:` field in `board_info.yaml` must exactly match the board folder name.

---

## 5. The five files

| File | Required? | Purpose |
|---|---|---|
| `board_info.yaml` | yes | Board metadata (name, chip, description, manufacturer) |
| `board_peripherals.yaml` | yes | Low-level bus configurations (I2C, SPI, RMT, I2S, GPIO, UART) |
| `board_devices.yaml` | yes | High-level device configurations that consume the peripherals |
| `sdkconfig.defaults.board` | recommended | Board-specific Kconfig defaults (flash size, PSRAM, etc.) |
| `setup_device.c` | only if you have `type: custom` devices | Init/deinit functions for non-supported device types |

Do **not** include in the board directory:

- `CMakeLists.txt` — the parent build system handles the board's source files
- `idf_component.yml` — dependencies belong inline in `board_devices.yaml` under each device's `dependencies:` block
- `setup_device.h` — board-private types and constants live as `static` in `setup_device.c`
- `Kconfig.projbuild` — auto-generated by `idf.py bmgr` on first run

---

## 6. `board_info.yaml`

Four fields, no comments, no top-level `version:`:

```yaml
board: your_board_name
chip: esp32s3
description: "Manufacturer Product Name (16MB flash / 8MB OPI PSRAM)"
manufacturer: "Manufacturer"
```

**Convention notes:**

- `board:` must exactly match the board folder name.
- `chip:` is one of `esp32`, `esp32s2`, `esp32s3`, `esp32c3`, `esp32c6`, `esp32h2`, `esp32p4` (lowercase, no hyphens).
- `description:` is one terse sentence. Tim's metro and the breadboard both fit on one line. Include flash and PSRAM capacity in parentheses at the end.
- `manufacturer:` is the short single-word form: `"Adafruit"`, `"ESPRESSIF"`, `"M5Stack"`, `"DFRobot"`, `"LilyGo"`. Not `"Adafruit Industries"`.

---

## 7. `board_peripherals.yaml`

Top-level `version: 1.0.0`, then a `peripherals:` list. Each entry:

```yaml
- name: <type_prefix>_<role>
  type: <i2c|spi|rmt|i2s|gpio|uart|ledc>
  role: <master|slave|tx|rx|none>
  config:
    <key>: <value>
    ...
```

**Critical rule:** the YAML keys under `config:` must match the underlying ESP-IDF C struct field names exactly, with the same nesting. The generator translates YAML keys 1:1 into C struct initializers. If you guess wrong, the build fails with `field 'X' has no member` errors.

**Naming rule from customize_board.md:** "peripheral names **must start with the type**." So `i2c_master`, `spi_display`, `rmt_tx`, `gpio_a0`, `uart_debug`. Names are referenced by string from device entries, so once chosen, keep them stable.

### 5.1 I2C example

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

Note the **nested `pins:` block** with short field names `sda`/`scl`. Older `sda_io_num`/`scl_io_num` flat-style is wrong for ESP-Claw.

### 5.2 SPI example

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

Notes:

- ESP-Claw convention is **`SPI3_HOST`** (both breadboard and metro use it). SPI2 also works but is non-canonical.
- Omit MISO entirely if your bus is write-only (display). For general-purpose buses, use `data1_io_num: <gpio>` (new ESP-IDF SPI master driver) or `miso_io_num: <gpio>` (legacy field name) — if the build emits a field-mismatch error on one, switch to the other.
- `max_transfer_sz` defaults to 4092 (one DMA block) if omitted. Sized for displays, use `width × 2 × N_lines`.

### 5.3 RMT (NeoPixel) example

This is the canonical NeoPixel pattern, byte-identical between the breadboard and metro:

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

Note `clk_src` uses the **string form of the IDF enum** (`RMT_CLK_SRC_DEFAULT`), not the integer value.

### 5.4 GPIO example

```yaml
- name: gpio_user_button
  type: gpio
  role: none
  config:
    gpio_num: 4
    pull_mode: PULLUP             # PULLUP / PULLDOWN / FLOATING
```

### 5.5 UART example

```yaml
- name: uart_debug
  type: uart
  role: tx
  config:
    port: 0
    tx_io_num: 43
    rx_io_num: 44
    baud_rate: 115200
```

### 5.6 I2S example (PDM-out, from breadboard)

```yaml
- name: i2s_audio_out
  type: i2s
  role: master
  format: pdm-out                 # peer field of role/version
  config:
    sample_rate_hz: 16000
    clk_src: I2S_CLK_SRC_DEFAULT
    mclk_multiple: 256
    bclk_div: 8
    data_bit_width: 16
    slot_bit_width: I2S_SLOT_BIT_WIDTH_AUTO
    slot_mode: I2S_SLOT_MODE_MONO
    slot_mask: I2S_PDM_SLOT_LEFT
    data_fmt: "I2S_PDM_DATA_FMT_PCM"
    line_mode: I2S_PDM_TX_ONE_LINE_CODEC
    pins:
      dout: 9
    invert_flags:
      clk_inv: false
```

`format:` is a peer field of `role`, not nested under `config:`. The breadboard uses I2S for the speaker output pin.

### 5.7 LEDC example

```yaml
- name: ledc_backlight
  type: ledc
  role: none
  config:
    gpio_num: 16
    freq_hz: 5000
    duty: 0
    duty_resolution: LEDC_TIMER_10_BIT
    output_invert: true
```

---

## 8. `board_devices.yaml`

Top-level `version: 1.0.0`, then a `devices:` list. Each entry:

```yaml
- name: <device_name>
  chip: <chip_part_number>
  type: <device_type>
  sub_type: <subtype>             # only for type:display_lcd, type:fs_fat, etc.
  version: default
  init_skip: false                # optional; default false
  dependencies:                   # optional
    espressif/<component>:
      version: "<semver>"
      public: true
  config:
    <fields>
  peripherals:                    # omit entirely if no peripherals
    - name: <peripheral_name>
```

### 6.1 Supported device types

These types have built-in implementations in `esp_board_manager` v0.5.3+ and don't need `setup_device.c` code beyond optional factory functions:

| `type` | Sub-types | Notes |
|---|---|---|
| `display_lcd` | `spi`, `dsi`, `parlio` | Intel-8080-style parallel for `parlio`; **NO RGB dot-clock** sub_type yet |
| `lcd_touch_i2c` | (none) | I2C touch panels |
| `audio_codec` | (none) | I2S audio codecs |
| `gpio_ctrl` | (none) | Simple GPIO control devices |
| `gpio_expander` | (none) | I2C GPIO expanders (AW9523, etc.) |
| `fs_fat` | `spi`, `sdmmc` | SD card or flash filesystem |
| `camera` | `dvp`, `mipi`, `usb` | Camera interfaces |

### 6.2 The `type: custom` escape hatch

When your hardware doesn't fit a supported type, use `type: custom`. The framework registers metadata about the device but delegates init/deinit to your `setup_device.c`. Two patterns:

**Pattern A — full custom device with init/deinit:**

```yaml
- name: my_chip
  chip: chip_part_number
  type: custom
  version: default
  config:
    i2c_addr: 0x34
    frequency: 400000
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

The framework links the dependency but doesn't call any init function. Application code instantiates the LED strip on demand. **No `CUSTOM_DEVICE_IMPLEMENT` needed for `init_skip: true` devices.**

### 6.3 Display device example (SPI panel, supported sub_type)

```yaml
- name: display_lcd
  chip: ili9341
  type: display_lcd
  sub_type: spi
  version: default
  dependencies:
    espressif/esp_lcd_ili9341: "*"
  config:
    mirror_x: false
    mirror_y: false
    swap_xy: false
    invert_color: false
    x_max: 240
    y_max: 320
    io_spi_config:
      cs_gpio_num: 10
      dc_gpio_num: 9
      spi_mode: 0
      pclk_hz: 40000000
      flags:
        sio_mode: false
    lcd_panel_config:
      reset_gpio_num: -1
      rgb_ele_order: LCD_RGB_ELEMENT_ORDER_BGR
      bits_per_pixel: 16
      data_endian: LCD_RGB_DATA_ENDIAN_BIG
      flags:
        reset_active_high: false
  peripherals:
    - name: spi_display
```

Notes:

- For `type: display_lcd`, the device name should always be `display_lcd` (not `tft`, `screen`, etc.). The display arbiter looks for this exact name.
- `x_max`/`y_max` are the canonical display-dimension fields. Not `width`/`height`, not `h_res`/`v_res`.
- `io_spi_config` and `lcd_panel_config` are nested blocks mirroring the ESP-IDF `esp_lcd_panel_io_spi_config_t` and `esp_lcd_panel_dev_config_t` C struct names.

### 6.4 Cross-device dependencies

If device A depends on device B being initialized first, list device B **first** in `board_devices.yaml`. The framework initializes devices in YAML order. Device A can pull device B's handle via:

```c
some_handle_t *b_handle = NULL;
esp_board_device_get_handle("device_b_name", (void **)&b_handle);
```

---

## 9. `sdkconfig.defaults.board`

This file should contain **only board-hardware-specific Kconfig settings**. WiFi defaults, stack sizes, console settings, FreeRTOS knobs, and other project-wide defaults belong in the project's top-level `sdkconfig.defaults`, not here.

Tim's metro_esp32s3 file is the canonical minimum:

```
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_240=y
# Adafruit Metro ESP32-S3: 16MB QIO flash @ 80MHz
CONFIG_ESPTOOLPY_FLASHMODE_QIO=y
CONFIG_ESPTOOLPY_FLASHFREQ_80M=y
CONFIG_ESPTOOLPY_FLASHSIZE_16MB=y
CONFIG_PARTITION_TABLE_CUSTOM=y
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME="partitions_16MB.csv"
# 8MB OPI PSRAM @ 80MHz
CONFIG_SPIRAM=y
CONFIG_SPIRAM_MODE_OCT=y
CONFIG_SPIRAM_SPEED_80M=y
```

**What belongs here:**

- CPU frequency (`CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_*`)
- Flash size, mode, frequency
- Partition table reference (`partitions_16MB.csv` for 16MB, `partitions_8MB.csv` for 8MB)
- PSRAM presence and configuration (`CONFIG_SPIRAM=y`, `CONFIG_SPIRAM_MODE_OCT=y`, `CONFIG_SPIRAM_SPEED_*M=y`)
- Console interface IF non-default (e.g., `CONFIG_ESP_CONSOLE_USB_CDC=y` for boards with native USB instead of UART bridge)
- Board-specific peripheral support flags (e.g., `CONFIG_LCD_RGB_ISR_IRAM_SAFE=y` for RGB displays, `CONFIG_ESP_VIDEO_ENABLE_USB_UVC_VIDEO_DEVICE=y` for USB cameras)

**What does NOT belong here** (these are project defaults, not board-specific):

- WiFi enable flags
- Main task / timer task stack sizes
- FreeRTOS configuration knobs
- Memory placement defaults (BSS in PSRAM, etc.)
- Anything that's already a Kconfig default

### 7.1 Partition tables

ESP-Claw provides two partition tables in `application/edge_agent/`:

- `partitions_8MB.csv` — for boards with 8MB flash
- `partitions_16MB.csv` — for boards with 16MB flash (preferred)

The 16MB layout:

```
nvs       data nvs    0x9000  0x6000  WiFi credentials, settings
otadata   data ota    0xF000  0x2000  OTA update metadata
phy_init  data phy    0x11000 0x1000  WiFi PHY calibration
ota_0     app  ota_0          4M      Firmware partition A
ota_1     app  ota_1          4M      Firmware partition B
emote     data spiffs         3M      Animation/emote assets
storage   data fat            4M      Lua scripts, persistent state
```

The application mounts the `storage` partition as FAT at `/fatfs` for runtime use. Don't customize this unless you have a specific reason; the standard layout is what every other ESP-Claw board uses.

---

## 10. `setup_device.c`

Required only if you have `type: custom` devices with `init_skip: false` (which is the default). One file per board, located in the board directory.

### 8.1 Standard structure

```c
/*
 * SPDX-FileCopyrightText: 2026 <your name or org>
 * SPDX-License-Identifier: Apache-2.0
 */

#include <string.h>
#include "esp_log.h"
#include "esp_check.h"
#include "esp_board_manager_includes.h"   /* umbrella header */
#include "gen_board_device_custom.h"      /* auto-generated config types */
/* ... chip-specific driver headers ... */

static const char *TAG = "BOARD_NAME_SETUP_DEVICE";

/* ... static helpers and types ... */

static int my_chip_init(void *config, int cfg_size, void **device_handle) {
    dev_custom_my_chip_config_t *cfg = (dev_custom_my_chip_config_t *)config;
    /* ... init logic ... */
    return ESP_OK;
}

static int my_chip_deinit(void *device_handle) {
    /* ... cleanup ... */
    return ESP_OK;
}

CUSTOM_DEVICE_IMPLEMENT(my_chip, my_chip_init, my_chip_deinit);
```

### 8.2 Required includes

```c
#include "esp_board_manager_includes.h"   /* MANDATORY — umbrella for board-mgr APIs */
#include "gen_board_device_custom.h"      /* MANDATORY — auto-generated struct defs */
```

Do **not** include `esp_board_device.h` or `esp_board_periph.h` directly — use the umbrella. (The cores3 reference uses individual includes; ESP-Claw uses the umbrella.)

### 8.3 The `CUSTOM_DEVICE_IMPLEMENT` macro

```c
CUSTOM_DEVICE_IMPLEMENT(<yaml_device_name>, <init_func>, <deinit_func>);
```

The first argument is the **YAML `name:` field**, NOT a C identifier with quotes. The macro stringifies it. Init/deinit signatures are:

```c
int init_func(void *config, int cfg_size, void **device_handle);
int deinit_func(void *device_handle);
```

### 8.4 Auto-generated config struct field naming

The generator produces a struct named `dev_custom_<yaml_device_name>_config_t` containing one field per YAML key under `config:`, plus auto-populated metadata fields:

- `cfg->chip` — the YAML `chip:` value (string)
- `cfg->peripheral_name` — the first peripheral name from `peripherals:` (string)
- One field per `config:` YAML key

So this YAML:

```yaml
- name: my_chip
  chip: foo123
  type: custom
  config:
    i2c_addr: 0x34
    frequency: 400000
    custom_field_a: 42
  peripherals:
    - name: i2c_master
```

Produces this C struct:

```c
typedef struct {
    const char *chip;             /* "foo123" */
    const char *peripheral_name;  /* "i2c_master" */
    uint8_t i2c_addr;             /* 0x34 */
    uint32_t frequency;           /* 400000 */
    int32_t custom_field_a;       /* 42 */
} dev_custom_my_chip_config_t;
```

### 8.5 Getting peripheral and device handles

For peripheral handles (I2C bus, SPI bus, etc.):

```c
i2c_master_bus_handle_t bus = NULL;
esp_board_periph_get_handle(cfg->peripheral_name, (void **)&bus);
```

For other devices' handles (e.g., for cross-device init order):

```c
my_other_device_handle_t *other = NULL;
esp_board_device_get_handle("other_device_name", (void **)&other);
```

### 8.6 Display devices in `setup_device.c`

If your display uses a **supported sub_type** (`display_lcd:spi`, `display_lcd:dsi`, `display_lcd:parlio`) but with a chip the framework doesn't have a built-in factory for, provide one function with this exact name and signature:

```c
esp_err_t lcd_panel_factory_entry_t(esp_lcd_panel_io_handle_t io,
                                    const esp_lcd_panel_dev_config_t *panel_dev_config,
                                    esp_lcd_panel_handle_t *ret_panel)
{
    esp_lcd_panel_dev_config_t panel_dev_cfg = {0};
    memcpy(&panel_dev_cfg, panel_dev_config, sizeof(esp_lcd_panel_dev_config_t));
    /* set vendor_config if needed */
    return esp_lcd_new_panel_<your_chip>(io, &panel_dev_cfg, ret_panel);
}
```

The framework finds this function by name. **Do not register it via CUSTOM_DEVICE_IMPLEMENT.** No `static`. The framework handles bus setup, panel reset, panel init, etc. — you just create the panel object. Tim's metro_esp32s3 setup_device.c is 19 lines because that's all it does for an ILI9341.

### 8.7 Custom display devices (RGB dot-clock)

If your display uses RGB dot-clock interface (Adafruit Qualia, several P4 boards), you cannot use `type: display_lcd` because there's no RGB sub_type in v0.5.3. Use `type: custom` and return a `dev_display_lcd_handles_t *` from your init function:

```c
#include "dev_display_lcd.h"   /* dev_display_lcd_handles_t */

static int my_display_init(void *config, int cfg_size, void **device_handle) {
    dev_custom_display_lcd_config_t *cfg = (dev_custom_display_lcd_config_t *)config;
    dev_display_lcd_handles_t *h = calloc(1, sizeof(*h));

    esp_lcd_rgb_panel_config_t panel_cfg = { /* ... */ };
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

**Why return `dev_display_lcd_handles_t`?** Because ESP-Claw's `display_hal.c`, the display arbiter, and `lua_module_display` all expect that type from `esp_board_device_get_handle("display_lcd", ...)`. Returning a board-private struct breaks the rest of the framework's display handling. This is the same bridging pattern the breadboard uses for `audio_dac`/`audio_adc` (which return `dev_audio_codec_handles_t` even though their underlying hardware is USB UAC, not I2S).

For RGB displays specifically, `app_claw_ui_start()` must call `display_hal_create()` with `DISPLAY_HAL_PANEL_IF_RGB`. As of this writing it's unclear whether the existing detection logic recognizes `type: custom` displays correctly; if your boot splash doesn't appear but `esp_lcd_panel_draw_bitmap()` works directly, that's the issue and the fix is in `app_claw_ui_start()`, not the board files.

### 8.8 Initialization order

Devices initialize in YAML order. If device A's init function needs device B, list B first.

---

## 11. Build and verification

### 9.1 First build

```bash
cd /path/to/esp-claw/application/edge_agent
idf.py bmgr -b your_board_name        # generates components/gen_bmgr_codes/
idf.py build
```

The first command runs the YAML-to-C generator. It produces:

- `components/gen_bmgr_codes/gen_board_periph_handles.c`
- `components/gen_bmgr_codes/gen_board_periph_config.c`
- `components/gen_bmgr_codes/gen_board_device_handles.c`
- `components/gen_bmgr_codes/gen_board_device_config.c`
- `components/gen_bmgr_codes/gen_board_info.c`
- `components/gen_bmgr_codes/gen_board_device_custom.h`  ← types your setup_device.c casts to
- `components/gen_bmgr_codes/CMakeLists.txt`
- `components/gen_bmgr_codes/idf_component.yml`

If the generator fails, your YAML has a syntax issue or invalid field. Errors are explicit.

If `idf.py build` fails with "field X has no member" errors in `setup_device.c`, the auto-generated struct field names don't match what you wrote. Open `gen_board_device_custom.h` and adjust the field accesses in your `setup_device.c` to match.

### 9.2 First flash

```bash
idf.py -p <port> flash monitor
```

What to expect:

1. Boot log mentions `esp_board_manager` initializing each peripheral and device in order.
2. Each `setup_device.c` init function logs its progress (use `ESP_LOGI` liberally).
3. ESP-Claw prints WiFi provisioning status. If no WiFi is configured, the board enters AP mode with SSID like `ESP-Claw-XXXX`; connect and configure via the captive portal.
4. If you have a display, the boot splash and emote system should render.

### 9.3 Common first-boot symptoms

| Symptom | Likely cause |
|---|---|
| `gen_board_device_custom.h: no such file` | Forgot to run `idf.py bmgr -b <board_name>` first |
| `field 'X' has no member` in setup_device.c | YAML key/C field name mismatch — open the generated header |
| Display doesn't init | Reset GPIO wrong, or panel init bytes missing for non-trivial panels |
| Display inits but shows wrong colors | RGB-565 data line ordering wrong — try swapping byte order |
| Display flickers | `pclk_hz` too high for this panel; drop until stable |
| Boot splash doesn't appear | `app_claw_ui_start()` doesn't recognize your display's `panel_if`; check ESP-Claw integration |
| WiFi captive portal won't load | Browser cache; try `192.168.4.1` directly |

---

## 12. Debugging tips

- **Prefer `init_skip: true` + application-side init** when a device is complex enough that you'd rather construct it lazily than at boot.
- **Use `ESP_LOGI` heavily in init functions.** Boot logs are how you debug board bring-up.
- **Verify GPIO assignments against an authoritative source.** Check for an ESP-IDF-native board support component first (in IDF examples, the IDF Component Registry, or `esp_board_manager`'s own boards directory). If none exists, fall back to whichever framework ports cover this board (Arduino BSP, CircuitPython, MicroPython, vendor SDK). Cross-check across ports when more than one is available. See §13 for the full workflow. Don't trust marketing pinout diagrams alone.
- **Set `CONFIG_LCD_RGB_ISR_IRAM_SAFE=y`** if you have an RGB display. Without it, flash operations during runtime will corrupt the display.
- **Enable `CONFIG_ESP_CONSOLE_USB_CDC=y`** for boards with native USB only (no UART-bridge chip). Without it, your serial console silently fails.
- **`idf.py bmgr` regenerates the `gen_bmgr_codes/` directory.** Run it after every YAML change.

---

## 13. Verifying GPIO assignments

GPIO numbers are the most error-prone part of a board definition. Verify against multiple authoritative sources before generating the YAML.

### 13.1 Check for an ESP-IDF source first

Before consulting framework-specific ports (CircuitPython, Arduino), check whether an ESP-IDF-native source exists for the board:

**Espressif's own boards:** look in IDF examples, especially under `examples/<category>/<example>/main/idf_board_support/`. The ESP32-S3-EYE, ESP32-S3-LCD-EV-Board, ESP-BOX-3, and ESP32-P4 dev kits all have IDF-native pin definitions there.

**IDF Component Registry:** search `https://components.espressif.com/` for the manufacturer name. M5Stack, LilyGo, Waveshare, and Seeed publish board support components. If found, the component's `Kconfig` and headers are authoritative.

**`esp_board_manager` boards:** the YAMLs in `https://github.com/espressif/esp-gmf/tree/main/packages/esp_board_manager/boards/` are themselves IDF-native definitions for the boards they cover (cores3, dual_eyes, esp32_p4_function_ev, etc.). For these boards, your work is mostly to copy and adapt rather than starting fresh.

When an IDF-native source exists, **start there**. Use any framework port as a supporting source for chip-specific details (I/O expander bit positions, display init bytes) but trust the IDF source for raw GPIO numbers and peripheral host selections.

### 13.2 Framework ports — when no IDF source exists

Most boards from third-party manufacturers don't ship with IDF-native pin definitions. In that case, framework ports are the authoritative source. The two most widely-maintained ports are CircuitPython and the Arduino BSP, each with strengths:

**Arduino BSP variant** — most widely supported across vendors, broadest cross-referencing:
```
https://github.com/espressif/arduino-esp32/blob/master/variants/<board_name>/pins_arduino.h
```

The Arduino variants directory accepts contributions from many manufacturers (Adafruit, M5Stack, LilyGo, Seeed, DFRobot, Waveshare, etc.). It typically gives canonical macro names (`PIN_NEOPIXEL`, `PIN_I2C_POWER`, `LED_BUILTIN`) that match the manufacturer's documentation and forum support. The Arduino BSP also defines the canonical `Wire` and `SPI` bus pins (`PIN_WIRE_SDA`/`PIN_WIRE_SCL`/`MOSI`/`MISO`/`SCK`/`SS`). Many vendors ship Arduino library examples that verify the BSP works as written.

**CircuitPython port** (when one exists) — most complete for vendor-specific structures like I/O expanders and display init sequences:
```
https://github.com/adafruit/circuitpython/blob/main/ports/espressif/boards/<board_name>/pins.c
https://github.com/adafruit/circuitpython/blob/main/ports/espressif/boards/<board_name>/mpconfigboard.h
```

CircuitPython is most common for Adafruit boards but has community-contributed entries from other vendors as well. As one detailed example: on the Adafruit Qualia ESP32-S3 RGB-666, `pins.c` defines `board.TFT_PINS` (RGB display GPIOs), `board.TFT_IO_EXPANDER` (PCA9554A bit positions and init bytes), and the default bus pins; `mpconfigboard.h` defines the underlying `DEFAULT_I2C_BUS_SDA` macros and the `DOUBLE_TAP_PIN` for bootloader entry. Many other vendors expose similar structures in their CircuitPython entries. Read both files together — they reference each other.

**MicroPython port** (when one exists):
```
https://github.com/micropython/micropython/tree/master/ports/esp32/boards/<board_name>
```

MicroPython has community-driven coverage of many ESP32 boards and is often the only port maintained for boards from smaller vendors or hobbyist communities. The board directory typically contains:
- `mpconfigboard.h` — defines `MICROPY_HW_BOARD_NAME`, `MICROPY_HW_MCU_NAME`, GPIO macros for I2C/SPI/UART buses, and any board-specific build flags
- `mpconfigboard.cmake` — partition table reference, flash/PSRAM size and mode, partition layout
- `pins.csv` (on some boards) — a tabular GPIO-to-name mapping that's easy to read
- `manifest.py` — frozen Python modules; not directly relevant but sometimes hints at which onboard hardware the firmware activates

For boards where MicroPython is the only port available, its `mpconfigboard.h` and `mpconfigboard.cmake` are authoritative for GPIO assignments and memory configuration. The `pins.csv` format (when present) is particularly easy for an LLM to parse — each line is `Name,GPIOnum`. MicroPython tends to be less rich than CircuitPython for chip-specific helper structures (I/O expander bit dictionaries, display init sequences), but those details may live in vendor-published Python driver libraries on PyPI or GitHub instead.

**Vendor-specific SDKs** — some manufacturers maintain their own forks or SDK distributions. M5Stack publishes the M5Unified library, LilyGo publishes board-specific firmware repos. When these exist, they usually represent the manufacturer's verified-working configuration and should be consulted.

### 13.3 Cross-checking workflow

1. **If an IDF-native source exists**, start there. Generate the GPIO list from it. Skip to step 4.
2. Otherwise, generate the GPIO list from whichever framework port is most complete for this board (CircuitPython if it covers the board's specialized hardware; Arduino BSP otherwise; MicroPython as a fallback).
3. Cross-check each GPIO number against any other framework port that exists.
4. If sources disagree, check the commit dates — recent edits usually win.
5. If they're the same age or you can't tell, the schematic is the tiebreaker.
6. Also verify the board revision matches what you have in hand — some boards have `_revB` variants where one or two GPIOs moved.

### 13.4 What each source is best for

| Detail | Best source |
|---|---|
| Core GPIO assignments (display data lines, bus pins) | IDF-native if present; otherwise cross-check across framework ports |
| Peripheral host/DMA selection (which SPI controller, which I2S port) | IDF-native source; otherwise the manufacturer's reference firmware |
| Verified `sdkconfig.defaults` settings | IDF-native source; MicroPython `mpconfigboard.cmake` is also a verified source for flash/PSRAM/partition settings |
| Flash size, PSRAM mode/speed, partition table reference | IDF-native or MicroPython `mpconfigboard.cmake` |
| Tabular human-readable pin-name list | MicroPython `pins.csv` when present (easy to parse line-by-line) |
| I/O expander chip address and bit positions | CircuitPython tends to be most complete; check whichever port the manufacturer maintains |
| I/O expander init register values | Same — usually a CircuitPython init dict or vendor-specific helper |
| Display init byte sequences | Vendor's display-driver library (Python or C) in whichever port the manufacturer ships |
| Canonical macro names matching forum support | Arduino BSP (most widely cross-referenced) |
| Whether a pin is the bootloader DOUBLE_TAP pin | CircuitPython `mpconfigboard.h` if available; MicroPython `mpconfigboard.h` `MICROPY_HW_USB_VID`/`USB_PID` and reset-pin macros also informative; vendor docs otherwise |
| Verified pull-up/pull-down configuration on a GPIO | Arduino BSP `pinMode()` calls in example sketches |
| Default behavior of power switches (NeoPixel power, I2C power, etc.) | Arduino BSP (often defines `PIN_*_POWER` macros); MicroPython `mpconfigboard.h` macros for power-rail pins |
| Boards primarily covered by community/hobbyist contributors | MicroPython is often the only port available |

Note that framework ports often use different folder-naming conventions than ESP-Claw. CircuitPython and Arduino variants for Adafruit boards typically include the `adafruit_` prefix even though ESP-Claw's `adafruit/` namespace omits it from the board folder name. M5Stack's ports use a `m5stack_` prefix, LilyGo uses `lilygo_`, etc. Don't be confused by these differences — the same hardware has different folder names in different ports.

For boards from any vendor without an IDF-native source, the manufacturer's reference firmware or schematic is the authoritative source. Most major ESP32 board vendors (Adafruit, M5Stack, LilyGo, DFRobot, Seeed, Waveshare) maintain Arduino BSPs and/or PlatformIO platform definitions; some additionally publish CircuitPython or MicroPython ports. Use whichever exists. Don't rely on third-party tutorials or wiki pages — those are often stale or for different revisions of the same product.

---

## 14. Checklist before opening a PR

- [ ] Folder placed at `application/edge_agent/boards/<manufacturer>/<board_name>/`
- [ ] Folder name is lowercase, alphanumeric, underscores only
- [ ] `board:` field in `board_info.yaml` matches the folder name
- [ ] `manufacturer:` is the short single-word form
- [ ] `description:` is one terse sentence with `(NMB flash / NMB OPI/QIO PSRAM)` capacity hint
- [ ] `board_peripherals.yaml` and `board_devices.yaml` start with `version: 1.0.0`
- [ ] All peripheral names start with their type prefix (`i2c_*`, `spi_*`, `rmt_*`, `gpio_*`, `uart_*`, `i2s_*`, `ledc_*`)
- [ ] All YAML config keys match the underlying ESP-IDF C struct field names
- [ ] No `CMakeLists.txt`, `idf_component.yml`, `setup_device.h`, or `Kconfig.projbuild` in the board directory
- [ ] `sdkconfig.defaults.board` contains only board-hardware-specific items (~10-15 lines)
- [ ] Partition table reference uses `partitions_8MB.csv` or `partitions_16MB.csv`
- [ ] If `setup_device.c` is present, every `type: custom` device with `init_skip: false` has a matching `CUSTOM_DEVICE_IMPLEMENT(name, init, deinit)`
- [ ] All GPIO assignments verified against an authoritative source (ESP-IDF native if present, framework port like Arduino/CircuitPython/MicroPython, vendor SDK, or schematic)
- [ ] `idf.py bmgr -b <board_name>` succeeds without errors
- [ ] `idf.py build` completes
- [ ] First flash boots, ESP-Claw initializes, captive portal accessible
- [ ] If display present: boot splash renders correctly
- [ ] Apache-2.0 license headers on any C source files contributed
- [ ] Author's SPDX-FileCopyrightText line in every C source file

---

## 15. Reference boards in the repo

When in doubt, mirror an existing board:

| Board | Notes |
|---|---|
| `espressif/esp32_S3_DevKitC_1_breadboard` | Most complete reference: USB camera (`type: custom`), USB audio (`type: custom`), ST7789 SPI display (`type: display_lcd:spi`), NeoPixel (`type: custom`+`init_skip`) |
| `espressif/esp32_S3_DevKitC_1` | Bare DevKitC with no peripherals beyond defaults |
| `adafruit/metro_esp32s3` | Reference Adafruit board: ILI9341 SPI display, NeoPixel |
| `m5stack/m5stack_cores3` (in upstream `esp_board_manager`) | Full power-management chip example using `type: custom` |
| `lilygo/lilygo_t_display_s3` | Older naming convention |
| `dfrobot/dfrobot_k10` | Older naming convention |

---

## 16. Where to ask for help

- **`esp_board_manager` issues:** [github.com/espressif/esp-gmf](https://github.com/espressif/esp-gmf/issues) (the component lives in the GMF monorepo)
- **ESP-Claw issues:** [github.com/espressif/esp-claw/issues](https://github.com/espressif/esp-claw/issues)
- **Customize-board doc:** `esp-gmf/packages/esp_board_manager/docs/how_to_customize_board.md`
- **ESP-Claw docs:** [esp-claw.com](https://esp-claw.com/en/)

---

## 17. What this document doesn't cover

- **Adding new device types to `esp_board_manager`** (e.g., RGB dot-clock as a first-class display sub_type). That's an upstream contribution to `esp-gmf`, not ESP-Claw.
- **Adding new peripheral types** (same — upstream).
- **Modifying ESP-Claw application code** (display arbiter, Lua modules, etc.). Those are application-level concerns; this doc is strictly about board definitions.
- **CircuitPython or Arduino board support.** Those have their own conventions in their own repos.
- **Hardware bring-up** (schematic verification, soldering, PCB testing). This doc assumes the hardware works and you're describing it for software.
