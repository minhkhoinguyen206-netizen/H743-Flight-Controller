# Hardware notes

Design-level notes for both boards. For the signal-by-signal map see [`PINOUT.md`](PINOUT.md), for parts see [`BOM.md`](BOM.md). None of this has been checked against a populated board yet.

## Power tree

```
J_5V_IN (BEC, 5V)
   -> D1 Schottky (reverse protection)
   -> 5V_SYS
        -> U1 AP2112K-3.3 LDO (EN tied high, always on)
             -> 3V3_MAIN -> MCU, SD pull-ups, digital logic
                  -> FB1 ferrite bead -> 3V3_SENSOR -> IMU, barometer

J_SERVO_BEC (separate BEC, 5-8.4V) -> SERVO_V+ -> servo headers
   buffered by C4, 1000uF/16V

J_VBAT_SENSE -> R_BAT_TOP (100k) -> BAT_VOLT_ADC -> R_BAT_BOT (10k) -> GND
```

The servo rail is intentionally isolated from the FC's own 3.3 V and 5 V logic. A servo stalling and pulling current shouldn't be able to brown out the MCU — that's the whole reason for the separate BEC input rather than running everything off one rail.

The ferrite bead splitting `3V3_SENSOR` off `3V3_MAIN` is there to keep the IMU and barometer's analog-ish supply a bit quieter than the digital rail feeding the MCU, SD card, and USB.

Decoupling on the FC board: 10 µF + 100 nF on each rail at the regulator, plus a scatter of 100 nF / 4.7 µF near the MCU's VDD pins, and 2.2 µF on VCAP1/VCAP2 (which must only ever see GND, never 3.3 V — wiring those wrong is one of the easier mistakes to make and a good way to damage the internal core regulator).

## MCU — STM32H743VIT6

Cortex-M7, up to 480 MHz, 2 MB flash, 1 MB RAM, LQFP100.

HSE is a 16 MHz crystal with 18 pF load caps, routed as short as the layout allows — this matters more than it looks like it should, especially for clean USB timing.

BOOT0 has a 10k pull-down so it boots normally by default; the BOOT button pulls it to 3.3 V to force the system bootloader for DFU flashing. NRST has a 10k pull-up, a reset button to GND, and a 100 nF cap for debounce.

Programming is through `J_SWD`, a 5-pin 1.27 mm header (3V3, SWDIO, SWCLK, GND, NRST) — not 2.54 mm, easy to grab the wrong header for this one.

## Sensors

The IMU (ICM-42688-P) is on its own SPI bus, with both interrupt lines broken out (INT1/INT2) so firmware can run a tightly-timed loop off data-ready interrupts rather than polling. It's an LGA-14 part — fine to hand-solder with flux and a steady hand, but a hot air station or reflow oven makes it much less error-prone, and the same goes for the barometer.

The barometer (BMP390) sits on I2C1, CSB tied high for I2C mode, SDO tied low for address 0x76. The same I2C1 bus is also broken out on the GPS connector for an external compass, if the GPS module carries one.

## Storage

microSD in 4-bit SDMMC mode, for high-rate logging without bottlenecking on a 1-bit interface. The footprint is a Hirose DM3D-SF — see the BOM notes, this is one of the easier things to get the wrong part for.

## USB

USB-C wired as a full-speed device: VBUS into the protection diode and onto 5V_SYS, D+/D- through 22 Ω series resistors into PA12/PA11, and CC1/CC2 pulled down through 5.1k so it presents as a sink. Used for configuration, DFU flashing, and pulling logs off without removing the SD card.

## IO board

Six servo headers, each with a 100 Ω series resistor on the signal line, powered from the separate `SERVO_V+` rail. GPS, ELRS, and telemetry each get their own 4–6 pin connector with 5V/GND plus the relevant UART (and I2C, for GPS). The buzzer is a low-side 2N7002 switch; the LED output is a single data line behind a 100 Ω resistor for a WS2812-style strip. The safety switch and VBAT sense divider round out the rest of it.

This board does not carry motor/ESC outputs in this revision — see the note in PINOUT.md about the M1–M8/SPARE1–8 naming mismatch on the stack connector if you're trying to figure out where those pins went.

## Layout notes from review

A few things worth flagging from going back over the placement before this got routed:

- The stack connector needs to actually mate the two boards face-to-face — worth double-checking in 3D viewer that the connector ended up on the correct side of each board once it's all soldered, not just that the footprint is right.
- IMU placement away from the regulator and USB/high-current routing matters more than it might seem; keep it as close to board center as the layout allows.
- Keep the barometer away from the regulator's heat and from direct airflow once it's in an enclosure — a stray draft across the sensor will show up as noise on the altitude reading.
- Double check silkscreen text doesn't overlap between adjacent parts before sending Gerbers out — cleaning that up after fab is a lot more annoying than fixing it in the editor.
- Mounting hole spacing is 30.5 × 30.5 mm (M3, 3.2 mm holes) — worth confirming standoffs don't interfere with anything once the boards are stacked.

## Pre-power checklist

This is roughly the order things were meant to go in, expanded with what to expect at each step. Build incrementally — don't jump straight to a flight battery.

**Before anything is soldered:** check the bare board with a meter. 3V3_MAIN to GND, EXT_5V to GND, and SERVO_V+ to GND should all read open, not a short.

**Power section first** (regulator, diode, ferrite bead, power caps — no MCU yet): feed 5 V into the bench input header. `3V3_MAIN` and `3V3_SENSOR` should both read close to 3.3 V. If either one doesn't, stop and find out why before populating anything else.

**VCAP check:** confirm C7/C8 are wired only between VCAP1/VCAP2 and GND, never to 3V3 — worth a multimeter check before the MCU goes on, since this is hard to fix after the chip is soldered down.

**MCU:** solder the STM32, double-check pin 1 orientation, inspect under magnification for bridges before powering anything up.

**SWD:** connect an ST-Link to `J_SWD` and confirm the debugger actually sees the chip before doing anything else.

**USB-C:** before plugging into a computer, check CC1/CC2 read down to GND through their 5.1k resistors and that D+/D- aren't shorted to each other through the 22 Ω resistors.

**Crystal:** solder Y1 and the 18 pF caps, inspect closely for bridges — these pads are small and close together.

**Sensors:** only after the MCU and power rails are confirmed good — solder the IMU and barometer.

**microSD:** solder the socket and pull-ups, test card detect/read/write once firmware can talk to it.

**Stack connector:** before soldering, confirm pin 1 lines up between the two boards and that one side is the socket, the other the header — getting this backwards or rotated 180° will short rails that were never meant to touch.

**Servo rail:** feed the servo BEC in, check `SERVO_V+` reads correctly at all six servo headers' pin 2, and confirm none of the servo connectors are accidentally tied to 3V3 or EXT_5V instead.

**VBAT sense:** feed a known voltage into `J_VBAT_SENSE` and confirm the ADC pin reads roughly `VBAT × 10/110` — e.g. 12 V in should read close to 1.09 V.

**GPS/ELRS/telemetry:** wire up each module and confirm the FC's TX line is actually reaching that module's RX (this is where the net-name-vs-silkscreen-label flip described in PINOUT.md matters — re-check there if anything looks backwards).

**Final:** bench test servo response with linkages disconnected before any of this touches a control surface.
