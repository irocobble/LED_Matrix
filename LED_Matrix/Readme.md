\# STM32 8x8 LED Matrix Driver



A custom PCB driving a 64-LED (8x8) matrix display using an STM32F103C8T6 

(ARM Cortex-M3) microcontroller, with row/column multiplexing via MMBT2222A 

NPN transistors.



\## Features



\- \*\*MCU:\*\* STM32F103C8Tx (72MHz, Cortex-M3, 64KB Flash)

\- \*\*Display:\*\* 8x8 LED matrix (64 LEDs), row-multiplexed via 16x MMBT2222A 

&#x20; SMD transistors (8 row drivers, 8 column drivers)

\- \*\*Power:\*\* USB-C input (5V) with LM1117MP-ADJ regulator for onboard 3.3V rail

\- \*\*USB-UART Bridge:\*\* CH340G for serial programming/debug via USB-C, 

&#x20; with 12MHz active oscillator

\- \*\*Clock:\*\* 8MHz active SMD oscillator for STM32 system clock

\- \*\*Programming:\*\* UART bootloader (BOOT0 select) — no external ST-Link needed

\- \*\*Protection:\*\* USB CC resistors, polyfuse, ESD diodes, ferrite bead on VBUS

\- \*\*User Controls:\*\* Reset and BOOT0 tactile switches



\## Hardware Overview



| Block | Components |

|---|---|

| MCU | STM32F103C8Tx |

| Display Driver | 16x MMBT2222A (SOT-23) |

| USB-UART | CH340G + 12MHz oscillator |

| Power Regulation | LM1117MP-ADJ (3.3V), USB-C input |

| Clock | 8MHz active oscillator (ASE series) |

| Connectivity | USB-C (USB2.0, 16-pin receptacle) |



\## Design Tool



Designed in \*\*KiCad 9.0.3\*\*.



\## Status



🚧 Work in progress — PCB layout and firmware in development.



\## License



\[Choose a license, e.g., MIT]

