
# STM32 8×8 LED Matrix Driver



## AIM
To design a custom PCB that drives an 8×8 LED matrix (64 LEDs) using an 
STM32F103C8T6 microcontroller, using transistor-based multiplexing instead 
of a dedicated LED driver IC — and to use it as a stepping stone for 
learning custom STM32 PCB design end-to-end (schematic, layout, fab, bring-up).

---

## Design Path — What We Tried First

The first version of the schematic used **MMBT2222A (NPN)** transistors for 
both the row and column drivers. This works fine for the rows, which sink 
current to GND — a standard common-emitter low-side switch. It does **not** 
work cleanly for the columns.

The columns are high-side switches: the transistor emitter sits at +3.3V, 
and the GPIO has to pull the base *above* the emitter to turn an NPN on. 
Since the GPIO can only reach 3.3V — the same as the emitter — there's no 
headroom left to forward-bias the junction. The fix is to use the 
complementary part, **MMBT2907A (PNP)**, for the columns: base pulled low 
(toward GND) relative to the 3.3V emitter, which a GPIO can do easily. 
Rows stay NPN (MMBT2222A), columns switch to PNP (MMBT2907A).

A second early mistake was using **LM1117MP-ADJ** for the 3.3V rail without 
a feedback resistor divider on the ADJ pin — the adjustable variant needs 
that divider to set its output voltage. Switched to **LM1117-3.3**, the 
fixed-output version, which only needs IN/OUT/GND.

---

## How It Works — The Big Picture

**USB-C (5V)** → **LM1117-3.3 regulator** → **STM32F103C8T6** → **GPIO row/column scan** → **BJT drivers** → **8×8 LED matrix**

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
  and the load (sources current). NPN BJTs are suited to low-side switching 
  off the same rail as the GPIO; PNP BJTs are suited to high-side switching 
  for the same reason, in reverse.
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
* **Display:** 8×8 LED matrix (64 LEDs)
* **Matrix Drivers:** 16 × SOT-23 BJTs — 8× MMBT2907A (PNP, column high-side), 
  8× MMBT2222A (NPN, row low-side)
* **Power Input:** USB-C, 5V
* **Regulation:** LM1117-3.3 (fixed 3.3V LDO)
* **USB-UART Bridge:** CH340G with 12 MHz active oscillator
* **System Clock:** 8 MHz active oscillator
* **Programming:** UART bootloader via BOOT0 selection, no ST-Link required
* **Protection:** USB-C CC resistors, polyfuse, ESD diodes, ferrite bead on VBUS
* **Controls:** Reset and BOOT0 push buttons

| Module | Component |
|---|---|
| MCU | STM32F103C8T6 |
| Column drivers (high-side) | 8 × MMBT2907A (PNP, SOT-23) |
| Row drivers (low-side) | 8 × MMBT2222A (NPN, SOT-23) |
| USB-UART bridge | CH340G + 12 MHz oscillator |
| Regulator | LM1117-3.3 |
| System oscillator | 8 MHz active |
| USB connector | USB-C receptacle |

**Design tool:** KiCad 9.0.3

**Source files:** [link to your repo]

---

## Firmware — Planned Approach

* MCU scans rows/columns using a timer interrupt for consistent refresh rate
* Frame buffer in SRAM holds the current 8×8 bitmap; the scan routine only 
  reads from it, so updating the display is just writing to the buffer
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

* [x] Schematic design (Rev A — NPN-only columns, LM1117-ADJ)
* [x] Identified and fixed column driver and regulator issues (Rev B, in schematic)
* [ ] PCB routing
* [ ] Prototype fabrication
* [ ] Hardware validation
* [ ] Firmware development
* [ ] Display animation library

---

## Limitations and Future Work

The current design uses discrete BJTs for 16 drivers instead of a dedicated 
LED matrix driver IC (e.g. MAX7219), which is simpler to wire but uses more 
board space and GPIO pins than a driver-IC approach. No per-LED or per-column 
current-limiting resistors are present in the current schematic revision — 
these need to be added before fabrication to avoid overdriving the LEDs. 
Brightness control (PWM dimming) is not yet implemented in firmware.

## License
