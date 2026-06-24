# Bill of materials

Both boards' parts, pulled from the soldering BOM spreadsheets and cross-checked against the schematics. Footprints are written exactly as they appear in KiCad — match them, don't substitute "close enough" parts, especially for the connectors flagged below.

## FC board (`H743_FC_TOP`)

### ICs, diode, regulator
| Ref | Part | Footprint | Qty | Note |
|-----|------|-----------|----:|------|
| U2 | STM32H743VIT6 | `LQFP-100_14x14mm_P0.5mm` | 1 | check pin 1 orientation before soldering |
| U3 | ICM-42688-P | LGA-14 | 1 | exact LGA footprint, chip only — not a breakout |
| U4 | BMP390 | LGA-10 | 1 | chip only — not a breakout module |
| U1 | AP2112K-3.3 | `SOT-23-5` | 1 | VIN from 5V_SYS, VOUT to 3V3_MAIN |
| D1 | 1N5819 (S4) Schottky | `D_SOD-123` | 1 | watch polarity |
| FB1 | Ferrite bead | `L_0603_1608Metric` | 1 | 3V3_MAIN → 3V3_SENSOR |

### Crystal
| Ref | Value | Footprint | Qty |
|-----|-------|-----------|----:|
| Y1 | 16 MHz | `Crystal_SMD_3225-4Pin_3.2x2.5mm` | 1 |

### Connectors
| Ref | Part | Footprint | Qty | Note |
|-----|------|-----------|----:|------|
| J1 | 5V bench input | `PinHeader_1x02_P2.54mm_Vertical` | 1 | test input, not for a battery |
| J2 | USB-C 2.0 | `USB_C_Receptacle_HRO_TYPE-C-31-M-12` | 1 | must match exactly — many USB-C footprints look similar but aren't pin-compatible |
| J3 | microSD socket | `microSD_HC_Hirose_DM3D-SF` | 1 | confirm the actual part before buying — see sourcing notes |
| J4 | Stack connector, 2×20 1.27 mm | `PinSocket_2x20_P1.27mm_Vertical` | 1 | **female socket** on this board |
| J_SWD | SWD header | `PinHeader_1x05_P1.27mm_Vertical` | 1 | 3V3 / SWDIO / SWCLK / GND / NRST |

### Switches
| Ref | Function | Footprint | Qty |
|-----|----------|-----------|----:|
| SW1 | BOOT | `SW_SPST_B3U-1000P` (or equivalent) | 1 |
| SW2 | RESET | `SW_SPST_B3U-1000P` (or equivalent) | 1 |

### Resistors (0603)
| Value | Refs | Qty | Used for |
|-------|------|----:|----------|
| 10k | R1, R2, R7 | 3 | BOOT/RESET/IMU CS pull |
| 5.1k | R3, R4 | 2 | USB-C CC1/CC2 |
| 22 Ω | R5, R6, R10 | 3 | USB D+/D-, SD clock |
| 4.7k | R8, R9 | 2 | I2C pull-ups |
| 47k | R11–R15 | 5 | microSD CMD/D0–D3 pull-ups |

### Capacitors (0603 unless noted)
| Value | Refs | Qty | Note |
|-------|------|----:|------|
| 10 µF | C1, C3, C5 | 3 | power bulk |
| 100 nF | C2, C4, C6, C9, C10, C11, C12, C14, C17 | 9 | decoupling, keep close to VDD pins |
| 2.2 µF | C7, C8 | 2 | VCAP1/VCAP2 — GND only, never 3V3 |
| 4.7 µF | C13 | 1 | MCU bulk |
| 18 pF | C15, C16 | 2 | crystal load |
| 1 µF | C18 | 1 | sensor decoupling |

## IO board (`H743_IO_BOTTOM`)

### Active parts
| Ref | Part | Footprint | Qty | Note |
|-----|------|-----------|----:|------|
| Q1 | 2N7002 | `SOT-23` | 1 | buzzer low-side switch — check G/S/D against the footprint, not just the part outline |
| BZ1 | Buzzer, 5 V active | `Buzzer_12x9.5RM7.6` | 1 | + to EXT_5V, − to BUZZER_MINUS |
| SW1 | Safety switch | `SW_SPST_B3U-1000P` (or equivalent) | 1 | |

### Connectors
| Ref | Part | Footprint | Qty |
|-----|------|-----------|----:|
| J1, J2, J3, J5, J6, J7 | Servo, 1×03 right-angle | `PinHeader_1x03_P2.54mm_Horizontal` | 6 |
| J4 | Stack connector, 2×20 1.27 mm | `PinHeader_2x20_P1.27mm_Vertical` | 1 | **male header** on this board |
| J_GPS1 | GPS + compass | `PinHeader_1x06_P2.54mm_Vertical` | 1 |
| J_ELRS1 | ELRS RX | `PinHeader_1x04_P2.54mm_Vertical` | 1 |
| J_TELEM1 | Telemetry | `PinHeader_1x04_P2.54mm_Vertical` | 1 |
| J_LED1 | LED data out | `PinHeader_1x03_P2.54mm_Vertical` | 1 |
| J_5V_IN1 | 5V logic input | `PinHeader_1x02_P2.54mm_Vertical` | 1 |
| J_SERVO_BEC1 | Servo BEC input | `PinHeader_1x02_P2.54mm_Vertical` | 1 |
| J_VBAT_SENSE1 | Battery sense | `PinHeader_1x02_P2.54mm_Vertical` | 1 |

### Resistors (0603)
| Value | Refs | Qty | Used for |
|-------|------|----:|----------|
| 100 Ω | R1–R6 | 6 | servo signal series, S1–S6 |
| 100 Ω | R_LED1 | 1 | LED data series |
| 100k | R_BAT_TOP1 | 1 | VBAT divider, top |
| 10k | R_BAT_BOT1 | 1 | VBAT divider, bottom |
| 1k | R_BUZ_GATE1 | 1 | buzzer gate series |
| 100k | R_BUZ_PD1 | 1 | buzzer gate pull-down |
| 10k | R_SAFE1 | 1 | safety switch pull-up |

### Capacitors
| Ref | Value | Footprint | Qty | Note |
|-----|-------|-----------|----:|------|
| C1 | 100 µF | `Capacitor_SMD:CP_Elec_5x5.8` | 1 | polarized — check the + marking against silkscreen |
| C2 | 10 µF | `Capacitor_SMD:C_0805_2012Metric` | 1 | not polarized |
| C3, C5 | 100 nF | `Capacitor_SMD:C_0603_1608Metric` | 2 | not polarized |
| C4 | 1000 µF / 16 V | `Capacitor_THT:CP_Radial_D8.0mm_P3.50mm` | 1 | polarized, servo rail reservoir — if it's too tall to fit under a stack, it's fine to bend leads and mount it off-board with wires |

## Sourcing notes

**microSD socket.** The footprint placed on the board is Hirose DM3D-SF, which is push-pull and mounts from the top. Some suppliers will offer DM3BT-DSF-PEJS instead — that's a push-push, reverse-mount part, and it is not pin-compatible despite being the same general Hirose family. KiCad has separate footprints for the two for exactly this reason. If a shop can't confirm DM3D-SF specifically, don't assume "close enough."

**BMP390.** Needs to be the bare LGA-10 chip. Breakout boards (GY-BMP390, Waveshare-style modules, anything with "module" in the name) are entire small PCBs and won't land on the U4 pads at all. If the raw chip isn't available, the realistic options are leaving U4 unpopulated for now (everything else can still be brought up and tested) or swapping the design to a barometer that's easier to get as a bare part.

**Stack connector.** One board needs the female socket, the other the male header, both 2×20 at 1.27 mm pitch. Buying two of the same gender means the boards can't be assembled — worth double-checking which one you're ordering for which board.

**Buzzer.** Must match `Buzzer_12x9.5RM7.6` — a 12×7.5 mm or 9×5 mm part won't match the hole spacing.

**USB-C.** The footprint is HRO's `TYPE-C-31-M-12`. USB-C footprints across different manufacturers are not all interchangeable even when they look similar in a datasheet thumbnail — confirm against the actual footprint, not just "USB-C 16-pin."

**Big capacitors.** C1 on the IO board is an SMD `5×5.8mm` package; C4 is through-hole, 8 mm diameter, 3.5 mm lead pitch. A 1000 µF cap with a 10 mm body or 5 mm pitch won't fit as drawn.

**Passives generally.** Buy a full strip/reel of each 0603 value rather than the exact count used — they're small enough to lose a few during reflow or hand soldering, and the cost difference between 5 and 50 is negligible.

## Shopping list — by value

**Capacitors:** 100 nF ×11 (both boards combined) · 10 µF 0603 ×3 · 10 µF 0805 ×1 · 2.2 µF ×2 · 4.7 µF ×1 · 18 pF ×2 · 1 µF ×1 · 100 µF SMD 5×5.8 ×1 · 1000 µF/16V THT D8/P3.5 ×1

**Resistors (0603):** 100 Ω ×7 · 22 Ω ×3 · 10k ×5 · 47k ×5 · 5.1k ×2 · 4.7k ×2 · 100k ×2 · 1k ×1 — get the 100k/10k pair for the VBAT divider in 1% tolerance if you care about reading battery voltage accurately

**Semiconductors:** STM32H743VIT6 ×1 · ICM-42688-P ×1 · BMP390 (bare LGA-10) ×1 · AP2112K-3.3 ×1 · 2N7002 ×1 · 1N5819 ×1 · ferrite bead 0603 ×1

**Other:** 16 MHz crystal (3225) ×1 · USB-C HRO TYPE-C-31-M-12 ×1 · microSD Hirose DM3D-SF ×1 · buzzer 12×9.5mm RM7.6 ×1 · push buttons (B3U or equivalent) ×3, buy a few spares

**Headers:** 2×20 1.27mm female socket ×1 (FC board) · 2×20 1.27mm male header ×1 (IO board) · 1×3 2.54mm right-angle ×6 (servos) · 1×5 1.27mm (SWD) · assorted 1×02/1×04/1×06 2.54mm straight headers for the rest, or a 1×40 strip cut to length

## Soldering order

1. Power section first: regulator, diode, ferrite bead, power caps. Apply 5 V and check the rails before anything else goes on.
2. VCAP caps — confirm they go to GND only.
3. MCU — check pin 1, inspect for bridges.
4. SWD header, then connect a debugger and confirm the chip is detected.
5. USB-C and its CC/series resistors.
6. Crystal and load caps.
7. IMU and barometer — only once the MCU and power are confirmed working.
8. microSD socket and pull-ups.
9. Stack connector — confirm pin 1 alignment and socket/header genders before soldering either side.
10. On the IO board: small SMD passives, then the MOSFET, then the safety switch, then the buzzer (watch polarity), then the two big capacitors (watch polarity), then the stack header, then the servo connectors, then GPS/ELRS/telemetry/LED/power headers last.
