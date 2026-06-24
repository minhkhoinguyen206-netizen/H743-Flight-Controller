# Hardware Design

Detailed design notes for the two-board **Flight Systems v1** H743 flight
controller. For the full signal map see [`PINOUT.md`](PINOUT.md); for parts see
[`BOM.md`](BOM.md).

> ⚠️ Design-stage documentation. Nothing here has been electrically tested.

---

## Overview

The FC is split into two stacked boards joined by a 40-pin, 1.27 mm
board-to-board connector:

- **FC / Compute board** (`H743_FC_TOP`) — MCU, sensors, storage, USB, clock,
  3.3 V regulation.
- **IO / Power board** (`H743_IO_BOTTOM`) — power input/distribution, VBAT sense,
  servo headers, comms ports, buzzer, LED, safety switch.

Splitting compute from I/O keeps the dense, high-speed digital section (H743,
SDMMC, USB, crystal) physically separate from the noisier, higher-current power
and servo section.

---

## Power tree

```
            J_5V_IN (BEC, 5 V)
                  │
                  ▼
              D1  ┤►  1N5819 Schottky  (reverse / back-feed protection)
                  │
               5V_SYS ──────────────► (5 V rail, also bused to stack as EXT_5V)
                  │
                  ▼
            U1 AP2112K-3.3  (LDO, EN tied high → always on)
                  │
               3V3_MAIN ─────────────► MCU VDD ×5, SD pull-ups, digital logic
                  │
                  ▼
              FB1 (ferrite bead)
                  │
               3V3_SENSOR ───────────► IMU + barometer (clean analog/sensor rail)


   J_SERVO_BEC (separate BEC) ──► SERVO_V+ ──► servo headers (buffered by C4 1000 µF)

   VBAT (J_VBAT_SENSE) ──[ R_BAT_TOP 100k ]──┬── BAT_VOLT_ADC ──► MCU ADC (PC0)
                                             │
                                       [ R_BAT_BOT 10k ]
                                             │
                                            GND
```

### Key decisions
- **Separate servo rail.** Servos are powered from their own BEC into
  `J_SERVO_BEC`, *not* from the FC's 3.3 V or logic 5 V. Servo stall/transient
  current never browns out the MCU. `C4` (1000 µF / 16 V) is the servo-rail bulk
  reservoir.
- **Reverse protection** on the 5 V logic input via Schottky `D1`.
- **Clean sensor rail.** A ferrite bead (`FB1`) isolates `3V3_SENSOR` (IMU, baro)
  from `3V3_MAIN` (digital), reducing digital noise into the analog sensors.
- **VBAT divider** = 100k / 10k = ÷11. The MCU never sees raw battery voltage;
  only the divided `BAT_VOLT_ADC`. Use 1% resistors for accurate readings.

### Decoupling (FC board)
| Rail | Caps |
|------|------|
| 5V_SYS | C1 10 µF, C2 100 nF |
| 3V3_MAIN | C3 10 µF, C4 100 nF, plus C9–C13 (100 nF / 4.7 µF) bulk near MCU |
| 3V3_SENSOR | C5 10 µF, C6 100 nF; IMU local C17 100 nF + C18 1 µF |
| MCU core | C7, C8 = 2.2 µF on VCAP1/VCAP2 |

---

## Compute — STM32H743VIT6 (U2)

- Cortex-M7 @ up to 480 MHz, 2 MB flash, 1 MB RAM, LQFP100.
- **HSE:** 16 MHz crystal `Y1` with 18 pF load caps (`C15`, `C16`) — required for
  reliable USB and high-speed clocking.
- **Boot/Reset:**
  - `BOOT0` (PB94 region) — 10k pull-down (`R1`) holds normal boot; `SW1`
    pulls it to 3V3 to enter the system DFU bootloader.
  - `NRST` — 10k pull-up (`R2`), `SW2` resets to GND, `C14` 100 nF debounce.
- **Programming:** `J_SWD1` exposes 3V3 / SWDIO / SWCLK / GND / NRST (1.27 mm).
- **Core caps:** `C7`/`C8` 2.2 µF on the internal LDO (VCAP1/VCAP2).

---

## Sensors

### IMU — ICM-42688-P (U3), SPI
A high-performance 6-axis (gyro + accel) IMU on a dedicated SPI bus:
CS / SCK / MISO / MOSI on PA4–PA7, with INT1/INT2 on PC4/PC5 for data-ready /
sync. Powered from `3V3_SENSOR`. `R7` 10k holds CS high; `C17`/`C18` decouple
locally. INT pins let firmware run a tightly-timed gyro loop.

### Barometer — BMP390 (U4), I2C
Bosch BMP390 on `I2C1` (shared bus). `CSB` is tied high → **I2C mode**; `SDO` is
tied to GND → 7-bit address **0x76**. `INT` is unused. Bus pull-ups `R8`/`R9`
4.7k to `3V3_SENSOR`. Provides barometric altitude for altitude hold/logging.

> The same I2C1 bus is broken out on the GPS connector for an **external compass**
> (common on GPS+mag modules). Mind the total bus capacitance and addresses.

---

## Storage — microSD (J3), SDMMC 4-bit

microSD in 4-bit SDMMC mode (CMD, CLK, D0–D3) for high-rate flight logging.
Pull-ups `R11–R15` (47k) on CMD/D0–D3, 22 R series (`R10`) on CLK for signal
integrity. **Footprint = Hirose DM3D-SF** — see the sourcing warning in
[`BOM.md`](BOM.md) (DM3BT is *not* a drop-in).

---

## USB — USB-C 2.0 (J2)

USB-C receptacle (HRO `TYPE-C-31-M-12`, 14-pin) wired as a **full-speed device**:
- `VBUS` → `USB_5V`
- `D-`/`D+` → PA11/PA12 through 22 R series resistors (`R6`/`R5`)
- `CC1`/`CC2` → 5.1k pull-downs (`R3`/`R4`) to present as a UFP/sink

Used for configuration, DFU flashing, and mass-storage access to logs.

---

## I/O board peripherals

| Block | Implementation |
|-------|----------------|
| **Servo outputs ×6** | `J1, J2, J3, J5, J6, J7` (1×03). 100 R series (`R1–R6`) on each signal line. Powered from `SERVO_V+`. |
| **GPS** | `J_GPS1` (1×06): 5V, GND, UART1 TX/RX, I2C1 SCL/SDA (for GPS + compass). |
| **ELRS RX** | `J_ELRS1` (1×04): 5V, GND, UART2 TX/RX. |
| **Telemetry** | `J_TELEM1` (1×04): 5V, GND, UART3 TX/RX. |
| **Buzzer** | `BZ1` switched low-side by `Q1` (2N7002). `R_BUZ_GATE1` 1k series, `R_BUZ_PD1` 100k pull-down. |
| **Addressable LED** | `J_LED1` (1×03): 5V, GND, `LED_DIN` with 100 R series (`R_LED1`) — drives a WS2812 strip. |
| **Safety switch** | `SW1` push button, `SAFETY_SW` with 10k pull-up (`R_SAFE1`). |
| **VBAT sense** | `R_BAT_TOP1` 100k / `R_BAT_BOT1` 10k divider → `BAT_VOLT_ADC`; `C5` 100 nF filter. |

Spare/expansion nets carried on the stack: `CAN_TX_SPARE` / `CAN_RX_SPARE`
(for a future CAN transceiver), `RSSI_SPARE`, `BAT_CURR_ADC` (for a current
sensor), and `BOOT0_SPARE`.

---

## Design rules (from KiCad project)

| Parameter | FC board | IO board |
|-----------|----------|----------|
| Clearance | 0.15 mm | 0.20 mm |
| Default track width | 0.20 mm | 0.20 mm |
| Via | 0.6 mm pad / 0.3 mm drill | 0.6 mm / 0.3 mm |
| Min via | 0.5 mm diameter / 0.1 mm annular | same |
| Min hole | 0.3 mm | same |

Both projects use a single **Default** net class. Teardrops are enabled.

> **Stackup / layers:** the PCB editor screenshots show inner copper layers in
> use (a multi-layer board, ≥4 layers). Confirm the exact stackup with your fab,
> and consider controlled impedance if you care about USB signal integrity.

---

## Pre-power bring-up checklist

> Build incrementally. **Do not** connect a flight battery on first power-up.

**Before any power:**
- [ ] Run KiCad **DRC** clean on both boards.
- [ ] Cross-check the netlist against the schematic (no auto-renumbered nets).
- [ ] Visually inspect for solder bridges, especially MCU/SDMMC/USB fine pitch.
- [ ] Confirm `J_STACK` orientation: one board uses a male header, the other a
      female socket — pin 1 must align.

**Power section first (no MCU/sensors yet):**
- [ ] Feed `J_5V_IN` from a current-limited bench supply at 5 V.
- [ ] Measure `3V3_MAIN` and `3V3_SENSOR` — both should be ~3.3 V, low ripple.
- [ ] Check no rail is shorted to GND and current draw is sane.

**MCU bring-up:**
- [ ] Populate MCU, crystal, BOOT/RESET, SWD, USB.
- [ ] Connect SWD (ST-Link) — confirm the target is detected.
- [ ] Flash a blink; confirm clock comes up on the 16 MHz HSE.
- [ ] Confirm USB enumerates (CDC test).

**Sensors & I/O:**
- [ ] Read IMU WHO_AM_I over SPI; read BMP390 chip ID over I2C.
- [ ] Mount/read a microSD card.
- [ ] Bring up UARTs (GPS/ELRS/telem), buzzer, LED, safety switch, VBAT ADC.

**System:**
- [ ] Bench-test servo outputs and attitude response **with linkages/props off**.
- [ ] Verify failsafe behaviour before any flight.
