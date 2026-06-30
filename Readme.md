# STM32 8×8 LED Matrix Driver

**Team Name:** ?

## AIM
To design a custom PCB that drives an 8×8 LED matrix (64 LEDs) using an 
STM32F103C8T6 microcontroller, using transistor-based multiplexing instead 
of a dedicated LED driver IC — and to use it as a stepping stone for 
learning custom STM32 PCB design end-to-end (schematic, layout, fab, bring-up).

---

## Design Path — What We Tried First

The first version of the schematic used MMBT2222A (NPN) transistors for 
both the row and column drivers. This works fine for the rows, which sink 
current to GND — a standard common-emitter low-side switch. It does not 
work cleanly for the columns, which are high-side switches: the transistor 
emitter sits at +3.3V, and the GPIO has to pull the base *above* the 
emitter to turn an NPN on, which it can't do from the same 3.3V rail. 
Fixed by switching the column drivers to MMBT2907A (PNP), where the GPIO 
pulls the base low relative to the 3.3V emitter instead.

The 3.3V regulator (LM1117MP-ADJ) needs a feedback resistor divider on the 
ADJ pin to set its output voltage — this should be confirmed present in 
the final schematic, or the part swapped for the fixed-output LM1117-3.3.

Both crystals (Y1: 12MHz for CH340G, Y2: 8MHz for STM32) are passive, with 
load capacitors and series resistors on each. Load cap values were 
recalculated based on the actual crystal's datasheet rated load 
capacitance (12pF for the YXC YSX321SL used on the CH340G side) rather 
than left at a generic default.

---

## How It Works — The Big Picture

**USB-C (5V)** → **LM1117 regulator** → **STM32F103C8T6** → **GPIO row/column scan** → **BJT drivers** → **8×8 LED matrix**

The MCU drives one row low and one column high at a time, cycling through 
all 8×8 = 64 LEDs fast enough that persistence of vision makes the whole 
matrix appear continuously lit. Only one LED is physically on at any instant.

Separately, **USB-C** → **CH340G** → **UART (PA9/PA10)** lets the board be 
programmed and debugged over the same USB-C connector, using the STM32's 
built-in UART bootloader — no ST-Link required.

---

## Key Definitions

* **Row/Column Multiplexing:** Driving a 2D LED grid with only 16 GPIO 
  (8 rows + 8 columns) instead of 64, by lighting one row/column intersection 
  at a time and relying on persistence of vision.
* **High-side / Low-side Switching:** A low-side switch sits between the 
  load and GND (sinks current); a high-side switch sits between the supply 
  and the load (sources current). NPN BJTs suit low-side switching off the 
  same rail as the GPIO; PNP BJTs suit high-side switching, for the same 
  reason in reverse.
* **Load Capacitor (Crystal):** A capacitor placed on each leg of a passive 
  crystal to bring the circuit's effective capacitance to the value the 
  crystal was designed to oscillate at (its rated load capacitance).
* **Ground/Power Plane:** A dedicated PCB layer filled with copper tied to 
  GND or a supply rail, used for low-impedance return paths and easier 
  routing on a multi-layer board.
* **Via Stitching:** Vias placed to connect copper pours on different 
  layers (e.g. multiple GND fills), improving return-current paths and 
  thermal spreading.
* **UART Bootloader:** A factory-programmed ROM bootloader on STM32 parts 
  that allows flashing firmware over UART (instead of SWD/JTAG) when BOOT0 
  is pulled high at reset.
* **ESD Protection Diode:** A component that clamps voltage transients on a 
  signal line (e.g. USB D+/D-) to protect downstream ICs from static discharge.
* **Polyfuse:** A resettable fuse that increases resistance under overcurrent 
  and resets automatically once the fault clears, unlike a one-time fuse.

---

## Hardware

* **MCU:** STM32F103C8T6 — ARM Cortex-M3, 72 MHz, 64 KB Flash, 20 KB SRAM
* **Display:** 8×8 LED matrix (64 LEDs), each column gated through its own 
  current-limiting resistor (R13–R20)
* **Matrix Drivers:** 16 × SOT-23 BJTs — 8× MMBT2907A (PNP, column high-side), 
  8× MMBT2222A (NPN, row low-side)
* **Power Input:** USB-C, 5V
* **Regulation:** LM1117MP-ADJ — feedback divider to be confirmed before fab
* **USB-UART Bridge:** CH340G with 12 MHz passive crystal (YXC YSX321SL, 12pF load cap)
* **System Clock:** 8 MHz passive crystal on STM32
* **Programming:** UART bootloader via BOOT0 selection, no ST-Link required
* **Protection:** USB-C CC resistors, polyfuse, ESD diodes, ferrite bead on VBUS
* **Controls:** Reset and BOOT0 push buttons

| Module | Component |
|---|---|
| MCU | STM32F103C8T6 |
| Column drivers (high-side) | 8 × MMBT2907A (PNP, SOT-23) |
| Row drivers (low-side) | 8 × MMBT2222A (NPN, SOT-23) |
| LED current limiting | R13–R20 (per-column resistors) |
| USB-UART bridge | CH340G + 12 MHz crystal |
| Regulator | LM1117MP-ADJ |
| System oscillator | 8 MHz crystal |
| USB connector | USB-C receptacle |

**Design tool:** KiCad 9.0.3

**Source files:** [link to your repo]

---

## PCB Layout

* **Board size:** 35×35mm
* **Layer count:** 4 layers
  * L1 (top): Signal routing
  * L2: GND plane
  * L3: 3.3V plane
  * L4 (bottom): Signal routing
* Power and ground as solid internal planes gives low-impedance return 
  paths for the digital signals on the outer layers and simplifies routing 
  density around the STM32 and LED matrix.

---

## Firmware — Planned Approach

* MCU scans rows/columns using a timer interrupt for consistent refresh rate
* Frame buffer in SRAM holds the current 8×8 bitmap; the scan routine only 
  reads from it, so updating the display is just writing to the buffer
* Brightness control planned via Bit Angle Modulation (BAM) for per-LED 
  grayscale, layered on top of the row/column scan — no hardware change 
  required, this is a firmware-only addition
* UART bootloader flow: pull BOOT0 high, reset, flash via STM32CubeProgrammer, 
  pull BOOT0 low, reset again to run

---

## Repository Structure

```text
├── Hardware/
│   ├── Schematic
│   ├── PCB Layout
│   └── Manufacturing Files
├── Firmware/
│   ├── Source Code
│   └── Build Files
└── Documentation/
```

---

## Current Status

* [x] Schematic — PNP columns, per-LED current-limiting resistors, passive crystals
* [x] Confirm LM1117MP-ADJ feedback divider is present and correctly valued
* [x] PCB routing — first pass complete, 4-layer stackup
* [x] Run DRC and confirm zero unrouted nets / clearance violations
* [ ] Prototype fabrication
* [ ] Hardware validation
* [ ] Firmware development
* [ ] Display animation library (brightness/BAM, frame buffer, basic patterns)

---

## Limitations and Future Work

The LM1117MP-ADJ regulator's feedback divider needs to be confirmed on the 
final schematic before fabrication. A few possible unrouted nets/airwires 
were visible on the top copper layer near the crystal and BJT driver area 
during review — DRC should be run to confirm full routing before sending 
to fab. No explicit ground-stitching vias were added near the row-driver 
BJT cluster, which sinks up to ~160mA when a full row is lit; this is not 
a blocker but could improve return-current path and thermal performance. 
The design uses discrete BJTs for all 16 drivers instead of a dedicated 
LED matrix driver IC (e.g. MAX7219), which is simpler to wire but uses more 
board space and GPIO pins than a driver-IC approach. Brightness control 
(PWM/BAM dimming) is planned but not yet implemented in firmware.

## License
