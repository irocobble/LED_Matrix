
# STM32 8×8 LED Matrix Driver

A custom PCB designed to drive an 8×8 LED matrix (64 LEDs) using the **STM32F103C8T6** microcontroller. The design uses transistor-based row and column multiplexing for efficient LED control while minimizing GPIO usage.

## Features

* **Microcontroller:** STM32F103C8T6 (ARM Cortex-M3, 72 MHz, 64 KB Flash, 20 KB SRAM)
* **Display:** 8×8 LED matrix (64 LEDs)
* **Display Driver:** 16 × MMBT2222A NPN transistors for row and column multiplexing
* **Power Supply:** USB Type-C (5 V input) with onboard 3.3 V regulation using LM1117MP-ADJ
* **USB-to-UART Interface:** CH340G with 12 MHz oscillator for programming and serial communication
* **System Clock:** External 8 MHz active oscillator
* **Programming Support:** UART bootloader via BOOT0 selection (no ST-Link required)
* **Protection Features:**

  * USB-C CC resistors
  * Resettable polyfuse
  * ESD protection diodes
  * Ferrite bead filtering on VBUS
* **User Controls:**

  * Reset push button
  * BOOT0 selection button

## Hardware Overview

| Module            | Components                 |
| ----------------- | -------------------------- |
| Microcontroller   | STM32F103C8T6              |
| LED Matrix Driver | 16 × MMBT2222A (SOT-23)    |
| USB-UART Bridge   | CH340G + 12 MHz oscillator |
| Power Regulation  | LM1117MP-ADJ (3.3 V)       |
| System Clock      | 8 MHz active oscillator    |
| USB Interface     | USB Type-C receptacle      |

## PCB Design

The schematic and PCB layout were designed using **KiCad 9.0.3**.

## Project Goals

* Drive an 8×8 LED matrix using efficient multiplexing techniques.
* Provide USB-based programming and debugging without requiring external programmers.
* Serve as a compact STM32 development platform for display-oriented projects.
* Explore low-cost embedded display hardware design.

## Repository Contents

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

## Current Status

🚧 **Work in Progress**

* [x] Schematic design
* [x] Component selection
* [ ] PCB routing
* [ ] Prototype fabrication
* [ ] Hardware validation
* [ ] Firmware development
* [ ] Display animation library

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.



