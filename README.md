# MCB1700 Media Center

A multi-threaded media center for the Keil **MCB1700** board (NXP **LPC1768**, ARM Cortex-M3), built on the **Keil RTX** real-time OS. It is my final project for COE 718 (Embedded Systems Design) at Toronto Metropolitan University.

**Status: in progress (Fall 2026).** The design is done and the groundwork from four labs is complete. Firmware is added here as each milestone is built and tested on the board.

## What it will do

From a joystick-driven main menu, the user picks one of three apps:

| App | What it does | Hardware it uses |
| --- | --- | --- |
| **Photo gallery** | Shows bitmap images one at a time; joystick left/right to browse | 320x240 colour LCD, joystick |
| **Audio player** | Streams audio from a PC over USB and plays it on the board; the potentiometer sets the volume | USB device (audio class), DAC + speaker, ADC |
| **Games** | Snake, plus Pong or Tic-Tac-Toe | LCD, joystick, RTX timers |

See the [project overview](docs/media-center/overview.md) and the [architecture](docs/media-center/architecture.md) for the thread and priority design.

## Groundwork: COE 718 labs

Each lab built a piece the media center needs. The write-ups explain what I did, what I measured, and what I would now do differently.

| Lab | Topic | Used in the media center for |
| --- | --- | --- |
| [1](docs/labs/lab1-toolchain-and-io.md) | Keil µVision, board bring-up, joystick, LEDs, ITM `printf` | Menu input, debug output without a UART |
| [2](docs/labs/lab2-bit-banding.md) | Register bit access: masking vs. bit-banding, `-O0` vs. `-O3`, disassembly | Fast, atomic GPIO access in timing-critical code |
| [3a](docs/labs/lab3a-round-robin.md) | RTX threads with round-robin scheduling, signals, a mutex | Splitting the app into threads that share data safely |
| [3b](docs/labs/lab3b-preemptive.md) | Pre-emptive priority scheduling | Keeping audio ahead of the UI |
| [4](docs/labs/lab4-timers-and-priority-inversion.md) | Virtual (software) timers, signal chains, priority inversion | Game ticks, avoiding inversion on the shared LCD |

## Hardware and tools

| | |
| --- | --- |
| Board | Keil MCB1700 |
| MCU | NXP LPC1768 (ARM Cortex-M3, 100 MHz, 512 KB flash, 64 KB SRAM) |
| IDE / toolchain | Keil µVision 5, Arm Compiler |
| RTOS | Keil RTX (CMSIS-RTOS v1 API) |
| Debugging | µVision debugger, RTX Event Viewer, Logic Analyzer, ITM `printf` |

## Repository layout

```text
docs/
  labs/           one write-up per lab
  media-center/   overview, architecture, roadmap
firmware/         media center source (added milestone by milestone)
```

## Academic note

This is coursework for COE 718, which is still running this term. To respect academic integrity:

- The lab write-ups are my own summaries. They include short excerpts of code I wrote, not complete lab solutions.
- Graded lab reports are not published here.
- Starter code and board-support files from Keil/Arm or the course keep their original copyright and are not redistributed.

I plan to publish the full lab code after the course ends.
