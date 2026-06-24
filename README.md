# H743 Flight Controller (Flight Systems v1)

A two-board flight controller built around the STM32H743, designed for a fixed-wing UAV with GPS waypoint navigation and direct servo control. Designed in KiCad 9.0.7.

The board is split into two stacked PCBs that connect through a 40-pin, 1.27 mm board-to-board header:

- **FC board** (`H743_FC_TOP`) — the MCU, IMU, barometer, microSD, USB-C, crystal, and 3.3 V regulation.
- **IO board** (`H743_IO_BOTTOM`) — power input, VBAT sensing, six servo outputs, GPS/ELRS/telemetry ports, buzzer, LED, and the safety switch.

This split keeps the dense digital section (MCU, SDMMC, USB, crystal) away from the noisier power and servo wiring.

## Current state

Both schematics are finished and both boards have been laid out and routed. The bare PCBs have come back from fabrication (photos below). Nothing has been assembled yet — no parts are soldered, the board has never been powered, and there's no firmware for it. Everything in this repo is the design as drawn, not as built. Before ordering parts or soldering anything, run DRC again and check the actual board against the BOM and pinout docs here, because none of it has been confirmed against a real, populated board yet.

## Photos

| Bare PCBs as fabricated | IO board, 3D render |
|---|---|
| ![Bare PCBs](docs/images/boards-bare-front.jpg) | ![IO board 3D](docs/images/io-board-3d.jpg) |

| FC board layout | IO board layout |
|---|---|
| ![FC layout](docs/images/fc-board-layout.jpg) | ![IO layout](docs/images/io-board-layout.jpg) |

Full schematics (large): [FC board](docs/images/schematic-fc-top.png) and [IO board](docs/images/schematic-io-bottom.png).

## What it's meant to do

The goal for this FC is a fixed-wing UAV that can:

- log altitude, attitude, and GPS position
- drive six servos directly off the IMU for stabilized flight surfaces
- take commands from the ground over a radio link, with telemetry back
- fly a GPS waypoint mission programmed in ahead of time, working out its own pitch/roll and throttle to get there

Those are software goals — the board gives you the sensors and I/O to do it, but the actual stabilization, navigation, and waypoint logic is firmware, and that's the harder half of this project. See the firmware section below.

## Specs

- MCU: STM32H743VIT6, Cortex-M7 @ 480 MHz, 2 MB flash, 1 MB RAM, LQFP100
- IMU: ICM-42688-P, 6-axis, on SPI
- Barometer: BMP390, on I2C1
- Storage: microSD, SDMMC 4-bit (Hirose DM3D-SF socket)
- USB: USB-C 2.0 full-speed, for config/DFU/log access
- Clock: 16 MHz crystal
- 6 servo outputs, with no motor/ESC outputs on this revision (see note below)
- UARTs for GPS, ELRS receiver, and telemetry radio, each with its own 5V/GND
- I2C1 also broken out to the GPS port, for an external compass
- Battery voltage sense through a 100k/10k divider
- Separate BEC input for the servo rail, kept off the FC's own 3.3 V supply
- Buzzer, addressable LED output, and a hardware safety switch
- 40-pin, 1.27 mm stack connector between the two boards
- 4× M3 mounting holes, 30.5 × 30.5 mm spacing

Full pin-by-pin mapping is in [`docs/PINOUT.md`](docs/PINOUT.md).

### About the motor outputs

Earlier drafts of this board had M1–M8 motor/ESC outputs broken out on the stack connector. That was dropped — this build uses six direct servo outputs only, no ESCs. On the FC board the stack pins are still silkscreened M1–M8 out of the MCU's timer pins, but on the IO board those same physical pins are just SPARE1–8 and aren't wired to anything. If you want motor outputs on a future revision, that's the connector to use, but as it stands now, don't expect anything on those pins.

## Power

Logic power comes in through `J_5V_IN` as 5 V from a BEC, passes through a Schottky diode for reverse protection, and feeds an AP2112K-3.3 LDO. The IMU and barometer get their own 3.3 V rail (`3V3_SENSOR`) split off after a ferrite bead, mostly to keep MCU/USB/SD switching noise away from the sensors.

Servos are powered separately. `J_SERVO_BEC` takes input from its own BEC (5–8.4 V depending on what you use) straight to the servo headers, buffered by a 1000 µF capacitor. This rail never touches the FC's 3.3 V or 5 V logic supply, so a servo stalling under load doesn't brown out the MCU.

Battery voltage is read through `J_VBAT_SENSE`, a 100k/10k divider into the ADC — don't connect a battery directly to that header expecting it to power anything, it's sense-only. With a 12 V pack, expect roughly 1.09 V at the ADC pin.

## Repo layout

```
README.md
docs/
  HARDWARE.md       subsystem-level design notes, power tree, bring-up order
  PINOUT.md         MCU pin map, stack connector, every external connector
  BOM.md            both boards' bill of materials, with sourcing notes
  images/           board photos and full-size schematics
hardware/
  H743_FC_TOP.kicad_pro
  H743_IO_BOTTOM.kicad_pro
```

Only the `.kicad_pro` project files are here right now. To make this buildable by anyone else, the `.kicad_sch` and `.kicad_pcb` files for both boards still need to go in `hardware/`, along with any custom footprints/symbols used. Gerbers can go in `hardware/production/` once you're ready to fab another run.

## Firmware

Realistically there are two routes here.

The sane one is to port an existing flight stack rather than write attitude control and GPS navigation from scratch — that's years of accumulated work in projects like ArduPilot or INAV, and not something worth re-deriving. The pin layout here already follows fairly standard autopilot conventions (dedicated UARTs for GPS/RX/telemetry, SPI IMU, I2C baro+compass, SDMMC logging, a VBAT divider, buzzer, LED, safety switch), so the practical path is writing a board/target definition for ArduPilot (Plane) or INAV rather than a full flight stack. Given the servo-only setup (no ESC outputs), Plane or INAV's fixed-wing support is the better fit over a multirotor-focused stack.

Bare-metal C++ with CubeIDE/HAL is fine for bring-up — blinking an LED, reading the IMU, writing to the SD card, getting USB enumerating — before you touch any of the flight-stack work. MicroPython runs on the H743 too and is fine for ground-side tooling, but it's not the right choice for the actual real-time control loop.

## Bring-up

Don't wire this straight to a flight battery the first time it's powered. Rough order:

1. Solder just the power section first (LDO, Schottky, decoupling) and confirm `3V3_MAIN` and `3V3_SENSOR` read clean 3.3 V before anything else goes on the board.
2. Populate the MCU, crystal, SWD header, and USB. Confirm SWD can see the chip and USB enumerates.
3. Add the IMU and barometer, then the microSD socket, then the IO board connectors.
4. Bring up firmware in the same order — clock, USB, IMU, barometer, SD, UARTs, then servo PWM.
5. Bench test servo response with the linkages disconnected before anything is mechanically connected to control surfaces.

The full checklist with expected readings at each step is in [`docs/HARDWARE.md`](docs/HARDWARE.md) and [`docs/BOM.md`](docs/BOM.md).

## BOM

Both boards' parts lists, with footprints and sourcing notes (parts were picked to be cheap and available in Vietnam), are in [`docs/BOM.md`](docs/BOM.md).

## Things that still need checking

This is the list of stuff most likely to bite if you go straight to ordering parts:

- **microSD footprint.** The board wants a Hirose DM3D-SF (push-pull, top-mount). DM3BT-DSF-PEJS is a different, non-interchangeable footprint — easy to get sent the wrong one if a supplier substitutes "the closest Hirose part."
- **BMP390 needs to be the bare LGA-10 chip**, not a breakout module. Modules won't land on the U4 footprint at all.
- **Stack connector pin 7–14 naming mismatch** between the two boards (M1–M8 vs SPARE1–8) — see the motor output note above. Worth double-checking before you wire anything to those pins expecting motor PWM.
- **Stack connector genders.** One side needs to be a female socket, the other a male header, both 2×20 at 1.27 mm pitch, and pin 1 has to line up — mark pin 1 clearly on both boards before soldering.
- **VBAT divider** is 100k/10k, fixed ÷11 ratio — check that suits your pack voltage and ADC reference, and use 1% resistors if you want the voltage reading to be trustworthy.
- **Servo BEC sizing.** The 1000 µF cap on the servo rail is a reservoir, not a substitute for a BEC sized for your actual servos' stall current.
- **GPS/ELRS/telemetry RX/TX silkscreen convention.** On those connectors, the net name follows the FC's point of view (`GPS_TX` is the FC transmitting) but the silkscreen label is written from the plugged-in device's point of view (so the pin carrying `GPS_TX` is labelled `RX`, because that's the GPS module's RX pin). It's intentional and makes wiring third-party modules less error-prone, but it's worth knowing before you start probing with a meter and getting confused about which way is "TX."
- General: nothing on this board has been powered up yet. A full DRC pass, a netlist check against the schematic, and a careful visual inspection before parts go on are all still owed.

## Roadmap

- Commit the actual `.kicad_sch` / `.kicad_pcb` files (and any custom libraries) so the project is buildable by someone other than me
- Add Gerbers/drill files and an assembly drawing
- Work through the list above before ordering the next batch of parts
- Assemble the first board, power section first
- Firmware bring-up in the order described above
- Pick ArduPilot Plane or INAV and write the board target
- Bench-test attitude stabilization with props and servo linkages disconnected
- Write up calibration and the first flight checklist once there's something to fly

## Disclaimer

Nothing here has been tested, flown, or validated in any way — treat it as a design in progress, not a finished product. Autopilot hardware that hasn't been bench- and field-tested can cause real damage or injury if it fails in the air. Don't fly over people, check your local UAV rules before flying (in Vietnam that's the CAAV's registration and airspace requirements), and don't trust any value or connection here without checking it against the actual board first.

## License

No license has been added yet, so default copyright applies — others can view this but not reuse it without asking. If you want this to actually be open hardware, add a `LICENSE` file; common combinations are CERN-OHL for the hardware plus MIT/Apache-2.0 for any firmware that ends up in this repo.
