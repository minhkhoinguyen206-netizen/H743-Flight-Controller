# Pinout

Full signal map for both boards, taken from the KiCad schematics plus the soldering/pinout notes that were worked out alongside them. Not yet checked against a populated board — treat this as the design intent, confirm against the real PCB before relying on it.

One thing worth knowing up front: on the IO board connectors (GPS, ELRS, telemetry), the **net name** in KiCad is written from the FC's point of view, but the **silkscreen label** is written from the point of view of whatever you're plugging in. So the pin carrying the net `GPS_TX` (the FC transmitting) is labelled `RX` on the silkscreen, because that's the GPS module's receive pin. It's intentional — it means you wire third-party modules straight across (their TX to the pin marked RX, etc.) without having to think about it — but it's confusing the first time you're probing with a meter and the label seems backwards from the net name.

## STM32H743VIT6 (U2, LQFP100)

### Servo PWM
| Signal | Pin | Port |
|--------|-----|------|
| S1 | 22 | PA0 |
| S2 | 23 | PA1 |
| S3 | 24 | PA2 |
| S4 | 25 | PA3 |
| S5 | 34 | PB0 |
| S6 | 35 | PB1 |

### Motor PWM (see note below — not wired on the IO board in this revision)
| Signal | Pin | Port |
|--------|-----|------|
| M1 | 39 | PE9 |
| M2 | 41 | PE11 |
| M3 | 43 | PE13 |
| M4 | 45 | PE15 |
| M5 | 59 | PD12 |
| M6 | 60 | PD13 |
| M7 | 61 | PD14 |
| M8 | 62 | PD15 |

These timer outputs are still routed to the stack connector from the FC board's side, but the IO board doesn't use them — it has six servo headers and nothing else. The stack pins are SPARE1–8 on the IO board's schematic. If a future revision adds ESC outputs, this is the connector to bring them out on; for now there's nothing there.

### IMU — ICM-42688-P, SPI
| Signal | Pin | Port |
|--------|-----|------|
| IMU_CS | 28 | PA4 |
| IMU_SCK | 29 | PA5 |
| IMU_MISO | 30 | PA6 |
| IMU_MOSI | 31 | PA7 |
| IMU_INT1 | 32 | PC4 |
| IMU_INT2 | 33 | PC5 |

### Barometer + external compass — I2C1
| Signal | Pin | Port |
|--------|-----|------|
| I2C1_SCL | 95 | PB8 |
| I2C1_SDA | 96 | PB9 |

Shared between the onboard BMP390 and whatever's wired to the GPS port's SCL/SDA pins.

### USB
| Signal | Pin | Port | Note |
|--------|-----|------|------|
| USB_DM | 70 | PA11 | 22 Ω series |
| USB_DP | 71 | PA12 | 22 Ω series |

### UARTs
| Function | TX | RX | Peripheral |
|----------|----|----|------------|
| GPS | PA9 (68) | PA10 (69) | USART1 |
| ELRS RX | PD5 (86) | PD6 (87) | USART2 |
| Telemetry | PD8 (55) | PD9 (56) | USART3 |

### microSD — SDMMC, 4-bit
| Signal | Pin | Port |
|--------|-----|------|
| SDMMC_CMD | 83 | PD2 |
| SDMMC_CK | 80 | PC12 |
| SDMMC_D0 | 65 | PC8 |
| SDMMC_D1 | 66 | PC9 |
| SDMMC_D2 | 78 | PC10 |
| SDMMC_D3 | 79 | PC11 |

47k pull-ups on CMD/D0–D3, 22 Ω series on CLK.

### ADC
| Signal | Pin | Port |
|--------|-----|------|
| BAT_VOLT_ADC | 15 | PC0 |
| BAT_CURR_ADC | 16 | PC1 |

### Other I/O
| Signal | Pin | Port |
|--------|-----|------|
| SAFETY_SW | 89 | PB3 |
| BUZZER | 90 | PB4 |
| LED_STRIP | 91 | PB5 |

### System
| Signal | Pin | Note |
|--------|-----|------|
| OSC_IN / OSC_OUT | 12 / 13 (PH0/PH1) | 16 MHz crystal |
| NRST | 14 | 10k pull-up, reset button, 100 nF debounce |
| BOOT0 | 94 | 10k pull-down, BOOT button to 3V3 |
| SWDIO / SWCLK | 72 (PA13) / 76 (PA14) | SWD header |
| VCAP1 / VCAP2 | 48 / 73 | 2.2 µF to GND only — never to 3V3 |
| VDD ×5 | 11, 27, 50, 75, 100 | 3V3_MAIN |
| VSS / VSSA | 10 / 19 | GND |

## J_SWD — programming header (FC board)

1×05, 1.27 mm pitch.

| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | 3V3_MAIN | 3V3 |
| 2 | SWDIO | DIO |
| 3 | SWCLK | CLK |
| 4 | GND | GND |
| 5 | NRST | RST |

## J1 / 5V_IN (FC board bench input)

| Pin | Net | Silkscreen | Note |
|-----|-----|------------|------|
| 1 | 5V_IN | +5V | Test/bench input. Don't put a battery on this. |
| 2 | GND | GND | |

## J4 — 40-pin stack connector

Top board uses a female socket, bottom board a male header, both 2×20 at 1.27 mm. Pin 1 should be marked with a triangle or dot on both boards' silkscreen so the stack can't go on rotated.

| Pin | Net | Silkscreen | Pin | Net | Silkscreen |
|----:|-----|------------|----:|-----|------------|
| 1 | GND | GND | 2 | GND | GND |
| 3 | EXT_5V | 5V | 4 | EXT_5V | 5V |
| 5 | 3V3_MAIN | 3V3 | 6 | BOOT0_SPARE | BOOT/SP |
| 7 | SPARE1 ¹ | SP1 | 8 | SPARE2 ¹ | SP2 |
| 9 | SPARE3 ¹ | SP3 | 10 | SPARE4 ¹ | SP4 |
| 11 | SPARE5 ¹ | SP5 | 12 | SPARE6 ¹ | SP6 |
| 13 | SPARE7 ¹ | SP7 | 14 | SPARE8 ¹ | SP8 |
| 15 | S1 | S1 | 16 | S2 | S2 |
| 17 | S3 | S3 | 18 | S4 | S4 |
| 19 | S5 | S5 | 20 | S6 | S6 |
| 21 | GPS_TX | GPS_TX | 22 | GPS_RX | GPS_RX |
| 23 | ELRS_TX | ELRS_TX | 24 | ELRS_RX | ELRS_RX |
| 25 | TELEM_TX | TELEM_TX | 26 | TELEM_RX | TELEM_RX |
| 27 | I2C1_SCL | SCL | 28 | I2C1_SDA | SDA |
| 29 | CAN_TX_SPARE | CAN_TX | 30 | CAN_RX_SPARE | CAN_RX |
| 31 | BAT_VOLT_ADC | BAT_V | 32 | BAT_CURR_ADC | BAT_I |
| 33 | BUZZER | BUZ | 34 | SAFETY_SW | SAFE |
| 35 | LED_STRIP | LED | 36 | RSSI_SPARE | RSSI |
| 37 | GND | GND | 38 | GND | GND |
| 39 | EXT_5V | 5V | 40 | EXT_5V | 5V |

¹ On the FC board's own schematic, these same physical pins are silkscreened M1–M8 (the MCU's motor timer pins) — see the motor PWM note above. The IO board doesn't wire them to anything, hence SPARE.

## IO board connectors

### J_GPS1 — 1×06
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | EXT_5V | 5V |
| 2 | GND | GND |
| 3 | GPS_TX | RX |
| 4 | GPS_RX | TX |
| 5 | I2C1_SCL | SCL |
| 6 | I2C1_SDA | SDA |

### J_ELRS1 — 1×04
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | EXT_5V | 5V |
| 2 | GND | GND |
| 3 | ELRS_TX | RX |
| 4 | ELRS_RX | TX |

### J_TELEM1 — 1×04
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | EXT_5V | 5V |
| 2 | GND | GND |
| 3 | TELEM_TX | RX |
| 4 | TELEM_RX | TX |

### Servo headers — J1, J2, J3, J5, J6, J7 (1×03 each, right-angle)

| Header | Channel |
|--------|---------|
| J1 | S1 |
| J2 | S2 |
| J3 | S3 |
| J5 | S4 |
| J6 | S5 |
| J7 | S6 |

| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | Sx_SERVO | S / SIG |
| 2 | SERVO_V+ | + |
| 3 | GND | − / GND |

### J_5V_IN1 — 1×02
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | EXT_5V | +5V |
| 2 | GND | GND |

Feeds the FC board through the stack. Don't put a battery on this directly.

### J_SERVO_BEC1 — 1×02
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | SERVO_V+ | S+ / SERVO+ |
| 2 | GND | GND |

Separate BEC, 5–8.4 V depending on what you're running. Never tie this to 3V3 or EXT_5V.

### J_VBAT_SENSE1 — 1×02
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | VBAT | VBAT+ |
| 2 | GND | GND |

Sense-only, through the 100k/10k divider. Don't wire a battery's positive lead anywhere else expecting it to power something — this header isn't a power input.

### J_LED1 — 1×03
| Pin | Net | Silkscreen |
|-----|-----|------------|
| 1 | EXT_5V | 5V |
| 2 | GND | GND |
| 3 | LED_DIN | DIN |

WS2812-style data out, behind a 100 Ω series resistor.

### Buzzer / safety switch (onboard, no connector)
The buzzer is switched low-side by a 2N7002: gate driven through a 1k resistor with a 100k pull-down, drain to the buzzer's minus pin, source to GND. The buzzer's plus pin goes straight to EXT_5V.

The safety switch pulls `SAFETY_SW` to GND when pressed; it's otherwise held high by a 10k pull-up to 3V3_MAIN.

## Mechanical

Both boards use 4× M3 mounting holes (3.2 mm), spaced 30.5 × 30.5 mm.
