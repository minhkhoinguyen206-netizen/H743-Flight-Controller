# Pinout Reference

Complete signal map for the **Flight Systems v1** H743 flight controller, taken
from the KiCad schematics (`H743_FC_TOP`, `H743_IO_BOTTOM`).

> ⚠️ Derived from the schematic only — **not verified against a built board.**
> Cross-check against the actual `.kicad_sch` / `.kicad_pcb` before trusting it.

---

## STM32H743VIT6 (U2, LQFP100) — pin assignments

### Servo PWM outputs
| Signal | MCU pin | Port | Likely timer |
|--------|---------|------|--------------|
| S1 | 22 | PA0 | TIM2/TIM5 CH1 |
| S2 | 23 | PA1 | TIM2/TIM5 CH2 |
| S3 | 24 | PA2 | TIM2/TIM5 CH3 |
| S4 | 25 | PA3 | TIM2/TIM5 CH4 |
| S5 | 34 | PB0 | TIM3 CH3 |
| S6 | 35 | PB1 | TIM3 CH4 |

### Motor PWM outputs
| Signal | MCU pin | Port | Likely timer |
|--------|---------|------|--------------|
| M1 | 39 | PE9  | TIM1 CH1 |
| M2 | 41 | PE11 | TIM1 CH2 |
| M3 | 43 | PE13 | TIM1 CH3 |
| M4 | 45 | PE15 | TIM1 CH4 |
| M5 | 59 | PD12 | TIM4 CH1 |
| M6 | 60 | PD13 | TIM4 CH2 |
| M7 | 61 | PD14 | TIM4 CH3 |
| M8 | 62 | PD15 | TIM4 CH4 |

### IMU — ICM-42688-P (SPI)
| Signal | MCU pin | Port |
|--------|---------|------|
| IMU_CS | 28 | PA4 |
| IMU_SCK | 29 | PA5 |
| IMU_MISO | 30 | PA6 |
| IMU_MOSI | 31 | PA7 |
| IMU_INT1 | 32 | PC4 |
| IMU_INT2 | 33 | PC5 |

### Barometer + external I2C — I2C1
| Signal | MCU pin | Port |
|--------|---------|------|
| I2C1_SCL | 95 | PB8 |
| I2C1_SDA | 96 | PB9 |

`I2C1` is shared by the onboard **BMP390** and any **external compass** wired to
the GPS connector.

### USB (USB-C, full-speed)
| Signal | MCU pin | Port | Note |
|--------|---------|------|------|
| USB_DM | 70 | PA11 | 22 R series (R6) |
| USB_DP | 71 | PA12 | 22 R series (R5) |

### UARTs
| Function | TX | RX | Peripheral |
|----------|----|----|------------|
| GPS | PA9 (68) | PA10 (69) | USART1 |
| ELRS RX | PD5 (86) | PD6 (87) | USART2 |
| Telemetry | PD8 (55) | PD9 (56) | USART3 |

### microSD — SDMMC (4-bit)
| Signal | MCU pin | Port |
|--------|---------|------|
| SDMMC_CMD | 83 | PD2 |
| SDMMC_CK | 80 | PC12 |
| SDMMC_D0 | 65 | PC8 |
| SDMMC_D1 | 66 | PC9 |
| SDMMC_D2 | 78 | PC10 |
| SDMMC_D3 | 79 | PC11 |

Pull-ups R11–R15 (47k) on CMD/D0–D3; 22 R series (R10) on CLK.

### ADC / sensing
| Signal | MCU pin | Port |
|--------|---------|------|
| BAT_VOLT_ADC | 15 | PC0 |
| BAT_CURR_ADC | 16 | PC1 |

### Misc I/O
| Signal | MCU pin | Port |
|--------|---------|------|
| SAFETY_SW | 89 | PB3 |
| BUZZER | 90 | PB4 |
| LED_STRIP | 91 | PB5 |

### System
| Signal | MCU pin | Note |
|--------|---------|------|
| OSC_IN | 12 (PH0) | 16 MHz crystal |
| OSC_OUT | 13 (PH1) | 16 MHz crystal |
| NRST | 14 | 10k pull-up (R2) + RESET button (SW2) + 100 nF (C14) |
| BOOT0 | 94 | 10k pull-down (R1) + BOOT button (SW1, to 3V3) |
| SWDIO | 72 (PA13) | SWD header |
| SWCLK | 76 (PA14) | SWD header |
| VCAP1 / VCAP2 | 48 / 73 | 2.2 µF each (C7/C8) |
| VDD ×5 | 11,27,50,75,100 | 3V3_MAIN |
| VDDA | 21 | (verify VREF+/VDDA handling) |
| VSS / VSSA | 10 / 19 | GND |

---

## 40-pin stack connector (J4)

`J_STACK_TOP` (FC board) ↔ `J_STACK_BOTTOM` (IO board), 2×20, 1.27 mm pitch.
Pins mate 1-to-1. **Net names differ on pins 7–14** between the two boards
(see note).

| Pin | Signal | Pin | Signal |
|----:|--------|----:|--------|
| 1 | GND | 2 | GND |
| 3 | EXT_5V | 4 | EXT_5V |
| 5 | 3V3_MAIN | 6 | BOOT0_SPARE |
| 7 | M1 ¹ | 8 | M2 ¹ |
| 9 | M3 ¹ | 10 | M4 ¹ |
| 11 | M5 ¹ | 12 | M6 ¹ |
| 13 | M7 ¹ | 14 | M8 ¹ |
| 15 | S1 | 16 | S2 |
| 17 | S3 | 18 | S4 |
| 19 | S5 | 20 | S6 |
| 21 | GPS_TX | 22 | GPS_RX |
| 23 | ELRS_TX | 24 | ELRS_RX |
| 25 | TELEM_TX | 26 | TELEM_RX |
| 27 | I2C1_SCL | 28 | I2C1_SDA |
| 29 | CAN_TX_SPARE | 30 | CAN_RX_SPARE |
| 31 | BAT_VOLT_ADC | 32 | BAT_CURR_ADC |
| 33 | BUZZER | 34 | SAFETY_SW |
| 35 | LED_STRIP | 36 | RSSI_SPARE |
| 37 | GND | 38 | GND |
| 39 | EXT_5V | 40 | EXT_5V |

> ¹ **Naming inconsistency:** the FC board labels pins 7–14 `M1–M8` (the MCU's
> motor timer outputs). The IO board labels the *same physical pins* `SPARE1–8`
> because this revision of the IO board doesn't route motor/ESC outputs (it's a
> servo-driven design). Physically they're the same nets — verify this matches
> your intent before assembly.

---

## External connectors (IO / bottom board)

### J_GPS1 — GPS + external compass (1×06)
| Pin | Signal |
|----:|--------|
| 1 | 5V (EXT_5V) |
| 2 | GND |
| 3 | GPS_TX (FC → connector TX) |
| 4 | GPS_RX |
| 5 | I2C1_SCL |
| 6 | I2C1_SDA |

### J_ELRS1 — ExpressLRS receiver (1×04)
| Pin | Signal |
|----:|--------|
| 1 | 5V |
| 2 | GND |
| 3 | ELRS_TX |
| 4 | ELRS_RX |

### J_TELEM1 — Telemetry radio (1×04)
| Pin | Signal |
|----:|--------|
| 1 | 5V |
| 2 | GND |
| 3 | TELEM_TX |
| 4 | TELEM_RX |

### Servo headers — J1, J2, J3, J5, J6, J7 (1×03 each)
Standard servo pinout. Signal line has a 100 R series resistor (R1–R6).

| Pin | Signal |
|----:|--------|
| 1 | Sx_SERVO (signal) |
| 2 | SERVO_V+ |
| 3 | GND |

| Header | Channel |
|--------|---------|
| J1 | S1 |
| J2 | S2 |
| J3 | S3 |
| J5 | S4 |
| J6 | S5 |
| J7 | S6 |

### Power & sense connectors
| Connector | Pins | Pinout |
|-----------|------|--------|
| `J_5V_IN1` | 1×02 | 5V logic input (EXT_5V), GND — from a BEC |
| `J_SERVO_BEC1` | 1×02 | SERVO_V+ (servo rail), GND — from a separate BEC |
| `J_VBAT_SENSE1` | 1×02 | VBAT, GND — battery + for voltage sensing |
| `J_LED1` | 1×03 | 5V, GND, LED_DIN (WS2812 data, 100 R series R_LED1) |

### Buzzer / safety (on-board)
| Block | Detail |
|-------|--------|
| Buzzer | BZ1 driven low-side by Q1 (2N7002). Gate: 1k series (R_BUZ_GATE1), 100k pull-down (R_BUZ_PD1). Signal = `BUZZER`. |
| Safety switch | SW1 push button, `SAFETY_SW` with 10k pull-up (R_SAFE1) to 3V3_MAIN. |

---

## SWD programming header (FC board)

`J_SWD1` — 1×05, **1.27 mm pitch** (confirm pitch before buying header).

| Pin | Signal |
|----:|--------|
| 1 | 3V3_MAIN |
| 2 | SWDIO |
| 3 | SWCLK |
| 4 | GND |
| 5 | NRST |
