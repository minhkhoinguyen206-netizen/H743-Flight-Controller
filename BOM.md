# Bill of Materials

Parts list for both boards of the **Flight Systems v1** H743 flight controller,
compiled from the KiCad schematics. The design targets components that are cheap
and locally available in Vietnam.

> ⚠️ **Verify every footprint against the actual PCB before ordering.** This BOM
> is from the schematic and has not been checked against a built board. Pay
> special attention to the [sourcing warnings](#sourcing-warnings-read-before-buying).

---

## FC / Compute board (`H743_FC_TOP`)

### Active components
| Ref | Part | Package / footprint | Qty | Notes |
|-----|------|---------------------|----:|-------|
| U2 | STM32H743VIT6 | LQFP100 | 1 | Main MCU |
| U3 | ICM-42688-P | LGA-14 | 1 | 6-axis IMU (SPI) |
| U4 | BMP390 | LGA-10 | 1 | Barometer (I2C) — **raw chip, not a module** |
| U1 | AP2112K-3.3 | SOT-23-5 | 1 | 3.3 V LDO |
| D1 | 1N5819 (S4) | SOD-123 | 1 | Schottky, reverse protection |
| Q1 | *(none on this board)* | — | — | buzzer FET is on the IO board |

### Crystal & ferrite
| Ref | Value | Package | Qty |
|-----|-------|---------|----:|
| Y1 | 16 MHz crystal | 3225 (4-pad SMD) | 1 |
| FB1 | Ferrite bead (BLM18-type) | 0603 | 1 |

### Connectors
| Ref | Part | Footprint | Qty |
|-----|------|-----------|----:|
| J2 | USB-C 2.0 receptacle | HRO `TYPE-C-31-M-12` (14P) | 1 |
| J3 | microSD socket | **Hirose DM3D-SF** | 1 |
| J4 | Board-to-board stack (top) | 2×20, 1.27 mm — **female socket** | 1 |
| J_SWD1 | SWD header | 1×05, **1.27 mm** pitch | 1 |
| J1 | 5 V bench input | 1×02, 2.54 mm | 1 |

### Buttons
| Ref | Function | Footprint | Qty |
|-----|----------|-----------|----:|
| SW1 | BOOT (→ DFU) | SMD push `SW_SPST_B3U-1000P` | 1 |
| SW2 | RESET | SMD push `SW_SPST_B3U-1000P` | 1 |

### Resistors (0603)
| Value | Refs | Qty |
|-------|------|----:|
| 10 k | R1, R2, R7 | 3 |
| 22 Ω | R5, R6, R10 | 3 |
| 47 k | R11, R12, R13, R14, R15 | 5 |
| 5.1 k | R3, R4 | 2 |
| 4.7 k | R8, R9 | 2 |

### Capacitors (0603 unless noted)
| Value | Refs | Qty |
|-------|------|----:|
| 100 nF | C2, C4, C6, C9, C10, C11, C12, C14, C17 | 9 |
| 10 µF | C1, C3, C5 | 3 |
| 2.2 µF | C7, C8 (VCAP) | 2 |
| 18 pF | C15, C16 (crystal load) | 2 |
| 4.7 µF | C13 | 1 |
| 1 µF | C18 | 1 |

---

## IO / Power board (`H743_IO_BOTTOM`)

### Active components
| Ref | Part | Package | Qty | Notes |
|-----|------|---------|----:|-------|
| Q1 | 2N7002 | SOT-23 | 1 | N-MOSFET, buzzer low-side driver |
| BZ1 | Buzzer 5 V | 12×9.5 mm `RM7.6` (TMB12A05) | 1 | Footprint `Buzzer_12x9.5RM7.6` |

### Connectors
| Ref | Part | Footprint | Qty |
|-----|------|-----------|----:|
| J1, J2, J3, J5, J6, J7 | Servo headers | 1×03, 2.54 mm (right-angle) | 6 |
| J4 | Board-to-board stack (bottom) | 2×20, 1.27 mm — **male header** | 1 |
| J_GPS1 | GPS + compass | 1×06, 2.54 mm | 1 |
| J_ELRS1 | ELRS RX | 1×04, 2.54 mm | 1 |
| J_TELEM1 | Telemetry | 1×04, 2.54 mm | 1 |
| J_LED1 | WS2812 out | 1×03, 2.54 mm | 1 |
| J_5V_IN1 | 5 V logic input | 1×02, 2.54 mm | 1 |
| J_SERVO_BEC1 | Servo BEC input | 1×02, 2.54 mm | 1 |
| J_VBAT_SENSE1 | Battery sense | 1×02, 2.54 mm | 1 |

### Button
| Ref | Function | Footprint | Qty |
|-----|----------|-----------|----:|
| SW1 | Safety switch | SMD push `SW_SPST_B3U-1000P` | 1 |

### Resistors (0603)
| Value | Refs | Qty | Notes |
|-------|------|----:|-------|
| 100 Ω | R1–R6, R_LED1 | 7 | servo signal + LED data series |
| 100 k | R_BAT_TOP1, R_BUZ_PD1 | 2 | use **1%** for R_BAT_TOP1 |
| 10 k | R_BAT_BOT1, R_SAFE1 | 2 | use **1%** for R_BAT_BOT1 |
| 1 k | R_BUZ_GATE1 | 1 | buzzer gate series |

### Capacitors
| Value | Refs | Footprint | Qty |
|-------|------|-----------|----:|
| 1000 µF / 16 V | C4 | THT radial `D8.0mm_P3.50mm` | 1 |
| 100 µF | C1 | SMD electrolytic `CP_Elec_5x5.8` | 1 |
| 10 µF | C2 | 0805 | 1 |
| 100 nF | C3, C5 | 0603 | 2 |

---

## Consolidated shopping summary

Buy a full reel/strip of the small passives — at 0603 they're easy to lose, and
the per-strip cost is trivial.

**ICs / semis:** STM32H743VIT6 ×1 · ICM-42688-P ×1 · BMP390 (LGA-10) ×1 ·
AP2112K-3.3 ×1 · 2N7002 ×1 · 1N5819 ×1 · ferrite bead 0603 ×1

**Clock / connectors / EM:** 16 MHz crystal (3225) ×1 · USB-C HRO TYPE-C-31-M-12 ×1 ·
microSD Hirose DM3D-SF ×1 · buzzer TMB12A05 ×1

**Headers:** 2×20 1.27 mm socket (top) ×1 · 2×20 1.27 mm header (bottom) ×1 ·
1×3 2.54 mm right-angle ×6 (servos) · 1×40 2.54 mm strip (cut for GPS/ELRS/TELEM/
LED/power/sense) · 1×5 1.27 mm (SWD)

**Buttons:** SMD push `B3U-1000P` ×3 (BOOT, RESET, SAFETY) — buy ~5

**Resistors 0603 (1 strip each):** 100 Ω · 22 Ω · 1 k · 4.7 k · 5.1 k · 10 k ·
47 k · 100 k *(get 10 k and 100 k in 1% for the VBAT divider)*

**Capacitors:** 100 nF 0603 (strip) · 10 µF (0603 + 0805) · 18 pF 0603 ×2 ·
2.2 µF 0603 ×2 · 4.7 µF 0603 ×1 · 1 µF 0603 ×1 · 100 µF SMD `5x5.8` ×1 ·
1000 µF/16 V THT `D8/P3.5` ×1

---

## Sourcing warnings (read before buying)

1. **microSD: DM3D-SF, not DM3BT.** The PCB footprint is Hirose **DM3D-SF**
   (push-pull, top-mount). **DM3BT-DSF-PEJS** (push-push, reverse) has a
   *different* footprint and is **not** a drop-in. If a shop only has DM3BT,
   either find the matching part or change the PCB footprint — don't force it.

2. **BMP390 = raw LGA-10 chip, not a module.** Breakout boards
   (GY-BMP390, Waveshare, etc.) are whole PCBs and **will not** solder onto
   footprint `U4`. If you can't get the raw chip, you can leave `U4` unpopulated
   and bring up MCU/IMU/GPS first, or change the PCB to a barometer that's easier
   to source locally.

3. **Stack connectors must be opposite genders.** Top board = **female socket**,
   bottom board = **male header**, both 2×20 1.27 mm. Two males (or two females)
   won't stack.

4. **Buzzer must match `Buzzer_12x9.5RM7.6`.** A 12×7.5 mm or 9×5 mm buzzer won't
   fit the holes/pitch.

5. **SWD header pitch.** `J_SWD1` is **1.27 mm**, not 2.54 mm — confirm before
   buying so you don't end up with a header that won't fit.

6. **Big electrolytics must match footprint.** `C1` = SMD `5×5.8 mm`; `C4` =
   THT `D8 mm, 3.5 mm pitch`. A 10 mm / 5 mm-pitch 1000 µF won't fit.

---

## Assembly tips

- Solder the **power section first** and confirm 3.3 V is clean **before**
  populating the MCU/sensors (see [`HARDWARE.md`](HARDWARE.md#pre-power-bring-up-checklist)).
- LGA parts (IMU, baro) and the LQFP100 MCU benefit from a stencil + hot-air or
  reflow. Hand-soldering is possible but harder for the LGAs.
- Use the silkscreen reference designators (`R1`, `C1`, …) on the board to place
  each part; match them to the tables above.
- Keep `C7`/`C8` (VCAP) close to the MCU as laid out — they stabilise the core LDO.
