# CircuitPython Library Reference
<!-- LLM context document — generated 2026-04-10 -->

**Generated:** April 10, 2026  
**Total libraries:** 268 (208 Adafruit + 60 Community)

---

## Sources

**Adafruit Bundle:**
```
https://github.com/adafruit/Adafruit_CircuitPython_Bundle/blob/master/circuitpython_library_list.md
```
**Community Bundle:**
```
https://github.com/adafruit/CircuitPython_Community_Bundle/blob/main/circuitpython_community_library_list.md
```
**Bundle download:** `https://circuitpython.org/libraries`

---

## Installation

### Via circup (recommended)
[circup](https://learn.adafruit.com/keep-your-circuitpython-libraries-on-devices-up-to-date-with-circup) runs on the host PC — no device network access needed.
```sh
pip install circup
circup install <import-name>   # install
circup update                   # update all
circup list                     # show installed
circup search <keyword>         # search
```

### Manual
Download the bundle zip from `https://circuitpython.org/libraries`, find the `.mpy` file/folder, copy to `CIRCUITPY/lib/`.

---

## Complete Library List (alphabetical)

*268 libraries · Adafruit = official Adafruit bundle · Community = community-contributed bundle*

| Library | Bundle | Category | Import name |
|---|---|---|---|
| 74HC595 | Adafruit | io | `adafruit_74hc595` |
| AD5245 | Community | io | `cedargrove_ad5245` |
| AD5293 | Community | io | `cedargrove_ad5293` |
| AD9833 | Community | io | `cedargrove_ad9833` |
| Adafruit Soundboard | Community | audio | `adafruit_soundboard` |
| ADS1x15 | Adafruit | sensor | `adafruit_ads1x15` |
| ADT7410 | Adafruit | sensor | `adafruit_adt7410` |
| ADXL34x | Adafruit | sensor | `adafruit_adxl34x` |
| AHTx0 | Adafruit | sensor | `adafruit_ahtx0` |
| AM2320 | Adafruit | sensor | `adafruit_am2320` |
| AMG88xx | Adafruit | sensor | `adafruit_amg88xx` |
| APDS9960 | Adafruit | sensor | `adafruit_apds9960` |
| AS3935 Lightning Detector | Community | sensor | `circuitpython_as3935` |
| AS5600 Rotary Position | Community | sensor | `circuitpython_as5600` |
| AS726x | Adafruit | sensor | `adafruit_as726x` |
| AS7341 | Adafruit | sensor | `adafruit_as7341` |
| AT24MAC EEPROM | Community | storage | `at24mac_eeprom` |
| AT42QT Acorn | Community | io | `at42qt_acorn` |
| AT42QT2120 | Community | io | `circuitpython_at42qt2120` |
| ATECC | Adafruit | io | `adafruit_atecc` |
| AW9523 | Adafruit | io | `adafruit_aw9523` |
| AXP192 | Community | motor | `circuitpython_axp192` |
| AXP2101 | Community | motor | `circuitpython_axp2101` |
| BD3491FS | Adafruit | audio | `adafruit_bd3491fs` |
| BH1750 | Adafruit | sensor | `adafruit_bh1750` |
| Bitmap Font | Adafruit | utility | `adafruit_bitmap_font` |
| BLE | Adafruit | wireless | `adafruit_ble` |
| BLE Adafruit Services | Adafruit | wireless | `adafruit_ble_adafruit` |
| BluefruitSPI | Adafruit | wireless | `adafruit_bluefruitspi` |
| Bluepad32 | Community | io | `bluepad32` |
| BMA220 | Community | sensor | `circuitpython_bma220` |
| BME280 | Adafruit | sensor | `adafruit_bme280` |
| BME680 | Adafruit | sensor | `adafruit_bme680` |
| BMP280 | Adafruit | sensor | `adafruit_bmp280` |
| BMP3XX | Adafruit | sensor | `adafruit_bmp3xx` |
| BNO055 | Adafruit | sensor | `adafruit_bno055` |
| BNO08X | Adafruit | sensor | `adafruit_bno08x` |
| BNO08X RVC | Adafruit | sensor | `adafruit_bno08x_rvc` |
| bteve BT815/6 | Community | display | `circuitpython_bteve` |
| Bus Device | Adafruit | utility | `adafruit_bus_device` |
| CAP1188 | Adafruit | io | `adafruit_cap1188` |
| CCS811 | Adafruit | sensor | `adafruit_ccs811` |
| CharLCD | Adafruit | display | `adafruit_charlcd` |
| CircuitPlayground | Adafruit | utility | `adafruit_circuitplayground` |
| CircuitPython Requests | Adafruit | wireless | `adafruit_requests` |
| CLUE | Adafruit | utility | `adafruit_clue` |
| ConnectionManager | Adafruit | utility | `adafruit_connection_manager` |
| Crickit | Adafruit | motor | `adafruit_crickit` |
| Debouncer | Adafruit | utility | `adafruit_debouncer` |
| DHT | Adafruit | sensor | `adafruit_dht` |
| Display Button | Adafruit | display | `adafruit_display_button` |
| Display Notify | Adafruit | display | `adafruit_display_notify` |
| Display Shapes | Adafruit | display | `adafruit_display_shapes` |
| Display Text | Adafruit | display | `adafruit_display_text` |
| DisplayIO GC9A01 | Community | display | `circuitpython_gc9a01` |
| DisplayIO GC9D01 | Community | display | `circuitpython_gc9d01` |
| DisplayIO ILI9163 | Community | display | `electronutlabs_ili9163` |
| DisplayIO SH1106 | Community | display | `circuitpython_sh1106` |
| DisplayIO SH1107 | Adafruit | display | `adafruit_displayio_sh1107` |
| DisplayIO SSD1305 | Adafruit | display | `adafruit_displayio_ssd1305` |
| DisplayIO SSD1306 | Adafruit | display | `adafruit_displayio_ssd1306` |
| DisplayIO ST7565 | Community | display | `circuitpython_displayio_st7565` |
| DotStar | Adafruit | display | `adafruit_dotstar` |
| DPS310 | Adafruit | sensor | `adafruit_dps310` |
| DRV2605 | Adafruit | motor | `adafruit_drv2605` |
| DRV8830 | Community | motor | `cedargrove_drv8830` |
| DS1307 | Adafruit | rtc | `adafruit_ds1307` |
| DS1841 | Adafruit | io | `adafruit_ds1841` |
| DS18X20 | Adafruit | sensor | `adafruit_ds18x20` |
| DS2413 | Adafruit | io | `adafruit_ds2413` |
| DS3231 | Adafruit | rtc | `adafruit_ds3231` |
| DS3502 | Adafruit | io | `adafruit_ds3502` |
| DymoScale | Adafruit | sensor | `adafruit_dymoscale` |
| Dynamixel | Community | motor | `hierophect_dynamixel` |
| EMC2101 | Adafruit | sensor | `adafruit_emc2101` |
| EPD | Adafruit | display | `adafruit_epd` |
| ESP ATcontrol | Adafruit | wireless | `adafruit_esp_atcontrol` |
| ESP32SPI | Adafruit | wireless | `adafruit_esp32spi` |
| Fingerprint | Adafruit | io | `adafruit_fingerprint` |
| FocalTouch | Adafruit | io | `adafruit_focaltouch` |
| FONA | Adafruit | wireless | `adafruit_fona` |
| FRAM | Adafruit | storage | `adafruit_fram` |
| FS3000 Air Velocity | Community | sensor | `circuitpython_fs3000` |
| FXAS21002C | Adafruit | sensor | `adafruit_fxas21002c` |
| FXOS8700 | Adafruit | sensor | `adafruit_fxos8700` |
| gpio_expander | Community | io | `circuitpython_gpio_expander` |
| GPS | Adafruit | sensor | `adafruit_gps` |
| GT911 Touchscreen | Community | io | `circuitpython_gt911` |
| HCSR04 | Adafruit | sensor | `adafruit_hcsr04` |
| HCSR04 | Community | sensor | `circuitpython_hcsr04` |
| HID | Adafruit | io | `adafruit_hid` |
| HT16K33 | Adafruit | display | `adafruit_ht16k33` |
| HTS221 | Adafruit | sensor | `adafruit_hts221` |
| HTU21D | Adafruit | sensor | `adafruit_htu21d` |
| HTU31D | Adafruit | sensor | `adafruit_htu31d` |
| HX711 Load Cell | Community | sensor | `circuitpython_hx711` |
| HX8357 | Adafruit | display | `adafruit_hx8357` |
| I2C Expanders | Community | io | `circuitpython_i2c_expanders` |
| i2cEncoderLibV21 | Community | io | `circuitpython_i2cencoderlibv21` |
| ICM20X | Adafruit | sensor | `adafruit_icm20x` |
| IL0373 | Adafruit | display | `adafruit_il0373` |
| IL0398 | Adafruit | display | `adafruit_il0398` |
| IL91874 | Adafruit | display | `adafruit_il91874` |
| ILI9341 | Adafruit | display | `adafruit_ili9341` |
| Image Load | Adafruit | utility | `adafruit_imageload` |
| INA219 | Adafruit | sensor | `adafruit_ina219` |
| INA260 | Adafruit | sensor | `adafruit_ina260` |
| INA3221 | Community | sensor | `circuitpython_ina3221` |
| IO HTTP | Adafruit | wireless | `adafruit_io` |
| IRRemote | Adafruit | io | `adafruit_irremote` |
| IS31FL3731 | Adafruit | display | `adafruit_is31fl3731` |
| JLed | Community | display | `jled` |
| L3GD20 | Adafruit | sensor | `adafruit_l3gd20` |
| Laser AT Rangefinder | Community | sensor | `circuitpython_laser_at` |
| Laser Egismos Rangefinder | Community | sensor | `circuitpython_laser_egismos` |
| LC709203F | Adafruit | sensor | `adafruit_lc709203f` |
| LED Animation | Adafruit | display | `adafruit_led_animation` |
| LIDARLite | Adafruit | sensor | `adafruit_lidarlite` |
| LILYGO T-Deck | Community | utility | `circuitpython_lilygo_t_deck` |
| LIS2DH12 | Community | sensor | `electronutlabs_lis2dh12` |
| LIS2MDL | Adafruit | sensor | `adafruit_lis2mdl` |
| LIS331 | Adafruit | sensor | `adafruit_lis331` |
| LIS3DH | Adafruit | sensor | `adafruit_lis3dh` |
| LIS3MDL | Adafruit | sensor | `adafruit_lis3mdl` |
| Logging | Adafruit | utility | `adafruit_logging` |
| LPS2X | Adafruit | sensor | `adafruit_lps2x` |
| LPS35HW | Adafruit | sensor | `adafruit_lps35hw` |
| LSM303 | Adafruit | sensor | `adafruit_lsm303` |
| LSM303 Accel | Adafruit | sensor | `adafruit_lsm303_accel` |
| LSM303DLH Mag | Adafruit | sensor | `adafruit_lsm303dlh_mag` |
| LSM6DS | Adafruit | sensor | `adafruit_lsm6ds` |
| LSM9DS0 | Adafruit | sensor | `adafruit_lsm9ds0` |
| LSM9DS1 | Adafruit | sensor | `adafruit_lsm9ds1` |
| LTC166X DAC | Community | io | `creativecontrol_ltc166x` |
| LTR329ALS01 | Community | sensor | `electronutlabs_ltr329als01` |
| LTR390 | Adafruit | sensor | `adafruit_ltr390` |
| M5Stack PbHub | Community | io | `circuitpython_m5stack_pbhub` |
| MatrixKeypad | Adafruit | io | `adafruit_matrixkeypad` |
| MAX31855 | Adafruit | sensor | `adafruit_max31855` |
| MAX31856 | Adafruit | sensor | `adafruit_max31856` |
| MAX31865 | Adafruit | sensor | `adafruit_max31865` |
| MAX7219 | Adafruit | display | `adafruit_max7219` |
| MAX9744 | Adafruit | audio | `adafruit_max9744` |
| MCP230xx | Adafruit | io | `adafruit_mcp230xx` |
| MCP2515 | Adafruit | io | `adafruit_mcp2515` |
| MCP3xxx | Adafruit | io | `adafruit_mcp3xxx` |
| MCP4725 | Adafruit | io | `adafruit_mcp4725` |
| MCP4728 | Adafruit | io | `adafruit_mcp4728` |
| MCP48XX DAC | Community | io | `circuitpython_mcp48xx` |
| MCP9600 | Adafruit | sensor | `adafruit_mcp9600` |
| MCP9808 | Adafruit | sensor | `adafruit_mcp9808` |
| MiniMQTT | Adafruit | wireless | `adafruit_minimqtt` |
| mitutoyo Digimatic | Community | sensor | `circuitpython_mitutoyo` |
| MLX90393 | Adafruit | sensor | `adafruit_mlx90393` |
| MLX90395 | Adafruit | sensor | `adafruit_mlx90395` |
| MLX90614 | Adafruit | sensor | `adafruit_mlx90614` |
| MLX90640 | Adafruit | sensor | `adafruit_mlx90640` |
| MMA8451 | Adafruit | sensor | `adafruit_mma8451` |
| MONSTERM4SK | Adafruit | display | `adafruit_monsterm4sk` |
| Motor | Adafruit | motor | `adafruit_motor` |
| MPL115A2 | Adafruit | sensor | `adafruit_mpl115a2` |
| MPL3115A2 | Adafruit | sensor | `adafruit_mpl3115a2` |
| MPR121 | Adafruit | io | `adafruit_mpr121` |
| MPRLS | Adafruit | sensor | `adafruit_mprls` |
| MPU6050 | Adafruit | sensor | `adafruit_mpu6050` |
| MPU6886 | Community | sensor | `circuitpython_mpu6886` |
| MS8607 | Adafruit | sensor | `adafruit_ms8607` |
| MSA301 | Adafruit | sensor | `adafruit_msa301` |
| NAU7802 ADC | Community | sensor | `cedargrove_nau7802` |
| NeoPixel | Adafruit | display | `neopixel` |
| NeoPixel SPI | Adafruit | display | `adafruit_neopixel_spi` |
| NeoTrellis | Adafruit | display | `adafruit_neotrellis` |
| NeoTrellisM4 Extended | Community | display | `circuitpython_trellism4_extended` |
| nRF24L01 | Community | wireless | `circuitpython_nrf24l01` |
| NTP | Adafruit | utility | `adafruit_ntp` |
| Nunchuk | Adafruit | io | `adafruit_nunchuk` |
| OV7670 | Adafruit | sensor | `adafruit_ov7670` |
| PAJ7620 Gesture | Community | sensor | `circuitpython_paj7620` |
| PCA9674 | Community | io | `circuitpython_pca9674` |
| PCA9685 | Adafruit | motor | `adafruit_pca9685` |
| PCA9955B LED Driver | Community | display | `circuitpython_pca9955b` |
| PCD8544 | Adafruit | display | `adafruit_pcd8544` |
| PCF8523 | Adafruit | rtc | `adafruit_pcf8523` |
| PCF8591 | Adafruit | io | `adafruit_pcf8591` |
| PCT2075 | Adafruit | sensor | `adafruit_pct2075` |
| Pixie | Adafruit | display | `adafruit_pixie` |
| PM25 | Adafruit | sensor | `adafruit_pm25` |
| PN532 | Adafruit | wireless | `adafruit_pn532` |
| PS2Controller | Community | io | `circuitpython_ps2controller` |
| PyBadger | Adafruit | utility | `adafruit_pybadger` |
| PyPortal | Adafruit | utility | `adafruit_pyportal` |
| QMI8658C IMU | Community | sensor | `circuitpython_qmi8658c` |
| RA8875 | Adafruit | display | `adafruit_ra8875` |
| Raspberry Pi Build HAT | Community | motor | `circuitpython_raspberrypi_buildhat` |
| RDA5807 FM Radio | Community | wireless | `circuitpython_rda5807` |
| Register | Adafruit | utility | `adafruit_register` |
| RFM69 | Adafruit | wireless | `adafruit_rfm69` |
| RFM9x | Adafruit | wireless | `adafruit_rfm9x` |
| RGB Display | Adafruit | display | `adafruit_rgb_display` |
| RM3100 Magnetometer | Community | sensor | `circuitpython_rm3100` |
| RockBlock | Adafruit | wireless | `adafruit_rockblock` |
| RPLIDAR | Adafruit | sensor | `adafruit_rplidar` |
| RuhRohRotaryIO | Community | io | `circuitpython_ruhrohrotaryio` |
| SCD30 | Adafruit | sensor | `adafruit_scd30` |
| SD | Adafruit | storage | `adafruit_sd` |
| SD Card | Adafruit | storage | `adafruit_sdcard` |
| Seeed XIAO nRF52840 | Community | utility | `circuitpython_seeed_xiao_nrf52840` |
| Seesaw | Adafruit | io | `adafruit_seesaw` |
| Serial Controlled Servo | Community | motor | `circuitpython_serial_controlled_servo` |
| SGP30 | Adafruit | sensor | `adafruit_sgp30` |
| SGP40 | Adafruit | sensor | `adafruit_sgp40` |
| SharpMemoryDisplay | Adafruit | display | `adafruit_sharpmemorydisplay` |
| SHT31D | Adafruit | sensor | `adafruit_sht31d` |
| SHT4x | Adafruit | sensor | `adafruit_sht4x` |
| SHTC3 | Adafruit | sensor | `adafruit_shtc3` |
| SI4713 | Adafruit | audio | `adafruit_si4713` |
| SI5351 | Adafruit | io | `adafruit_si5351` |
| SI7021 | Adafruit | sensor | `adafruit_si7021` |
| SimpleIO | Adafruit | utility | `adafruit_simpleio` |
| Slideshow | Adafruit | utility | `adafruit_slideshow` |
| SSD1305 | Adafruit | display | `adafruit_ssd1305` |
| SSD1306 | Adafruit | display | `adafruit_ssd1306` |
| SSD1322 | Adafruit | display | `adafruit_ssd1322` |
| SSD1325 | Adafruit | display | `adafruit_ssd1325` |
| SSD1327 | Adafruit | display | `adafruit_ssd1327` |
| SSD1331 | Adafruit | display | `adafruit_ssd1331` |
| SSD1351 | Adafruit | display | `adafruit_ssd1351` |
| SSD1608 | Adafruit | display | `adafruit_ssd1608` |
| SSD1675 | Adafruit | display | `adafruit_ssd1675` |
| SSD1680 | Adafruit | display | `adafruit_ssd1680` |
| SSD1681 | Adafruit | display | `adafruit_ssd1681` |
| ST7735 | Adafruit | display | `adafruit_st7735` |
| ST7735R | Adafruit | display | `adafruit_st7735r` |
| ST7789 | Adafruit | display | `adafruit_st7789` |
| STMPE610 | Adafruit | io | `adafruit_stmpe610` |
| TC74 | Adafruit | sensor | `adafruit_tc74` |
| TCA9548A | Adafruit | io | `adafruit_tca9548a` |
| TCA9555 | Community | io | `community_circuitpython_tca9555` |
| TCS34725 | Adafruit | sensor | `adafruit_tcs34725` |
| TFmini | Adafruit | sensor | `adafruit_tfmini` |
| Thermal Printer | Adafruit | utility | `adafruit_thermal_printer` |
| Thermistor | Adafruit | sensor | `adafruit_thermistor` |
| Ticks | Adafruit | utility | `adafruit_ticks` |
| TicStepper | Community | motor | `circuitpython_ticstepper` |
| TLA202X | Adafruit | sensor | `adafruit_tla202x` |
| TLC5947 | Adafruit | display | `adafruit_tlc5947` |
| TLC59711 | Adafruit | display | `adafruit_tlc59711` |
| TLV320AIC3204 Audio Codec | Community | audio | `circuitpython_tlv320aic3204` |
| TLV493D | Adafruit | sensor | `adafruit_tlv493d` |
| TMP006 | Adafruit | sensor | `adafruit_tmp006` |
| TMP007 | Adafruit | sensor | `adafruit_tmp007` |
| TMP117 | Adafruit | sensor | `adafruit_tmp117` |
| TMP75 | Community | sensor | `circuitpython_tmp75` |
| Touchscreen | Adafruit | io | `adafruit_touchscreen` |
| TPA2016 | Adafruit | audio | `adafruit_tpa2016` |
| TSL2561 | Adafruit | sensor | `adafruit_tsl2561` |
| TSL2591 | Adafruit | sensor | `adafruit_tsl2591` |
| TTP229 Touch | Community | io | `circuitpython_ttp229` |
| Typing Extensions | Adafruit | utility | `adafruit_typing` |
| VCNL4010 | Adafruit | sensor | `adafruit_vcnl4010` |
| VCNL4020 | Adafruit | sensor | `adafruit_vcnl4020` |
| VCNL4040 | Adafruit | sensor | `adafruit_vcnl4040` |
| VEML6070 | Adafruit | sensor | `adafruit_veml6070` |
| VEML7700 | Adafruit | sensor | `adafruit_veml7700` |
| VL53L0X | Adafruit | sensor | `adafruit_vl53l0x` |
| VL53L1X | Adafruit | sensor | `adafruit_vl53l1x` |
| VS1053 | Adafruit | audio | `adafruit_vs1053` |
| WiiChuck | Community | io | `circuitpython_wiichuck` |

---

## Key Dependencies (near-universal)

| Import name | Purpose |
|---|---|
| `adafruit_bus_device` | I2C/SPI device base classes — required by nearly all sensor/display drivers |
| `adafruit_register` | Hardware register abstraction — required by many drivers |
| `adafruit_requests` | HTTP(S) requests over WiFi |
| `adafruit_display_text` | Text rendering on displays |
| `adafruit_imageload` | BMP/PNG image loading for displayio |
| `neopixel` | NeoPixel / WS2812 LED control |
| `adafruit_io` | Adafruit IO cloud platform client |
| `adafruit_minimqtt` | MQTT client |

---

## Notes for LLMs

- When recommending a library, always specify the bundle (Adafruit or Community) so the user downloads the right zip.
- The import name is the filename in `CIRCUITPY/lib/` without `.mpy`. It is not always derivable from the repo name.
- Do **not** conflate CircuitPython library names with MicroPython module names.
- Bundle version must match CircuitPython firmware version; mismatches cause `ImportError`.
- Multi-file libraries (folders) must be copied as a whole directory, not just `__init__.mpy`.
- `adafruit_bus_device` and `adafruit_register` are required by the vast majority of drivers.
- `circup` is the canonical installation tool and handles both bundles.

---

*Generated: April 10, 2026*
*Adafruit bundle list: https://github.com/adafruit/Adafruit_CircuitPython_Bundle/blob/master/circuitpython_library_list.md*
*Community bundle list: https://github.com/adafruit/CircuitPython_Community_Bundle/blob/main/circuitpython_community_library_list.md*
*circup: https://learn.adafruit.com/keep-your-circuitpython-libraries-on-devices-up-to-date-with-circup*
