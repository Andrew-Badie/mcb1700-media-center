# Media center: project overview

## The user's view

The board starts on a **main menu** with three choices. The user moves between them with the joystick and presses **Select** to open one. Every app returns to the menu the same way.

```text
+----------------------------+
|        MEDIA CENTER        |
|                            |
|   > Photo Gallery          |
|     Audio Player           |
|     Games                  |
|                            |
|  Up/Down: move  Sel: open  |
+----------------------------+
```

## Photo gallery

- Shows one bitmap at a time on the 320x240 LCD. Joystick left/right moves to the previous or next image.
- Images are converted ahead of time from `.bmp` to C arrays (exported from GIMP as arrays of pixel data) and compiled into flash, because the board has no file system.
- Stretch goal: a slideshow mode that advances on a timer, and a row of thumbnails along the bottom.

## Audio player

- The board connects to a PC over USB as a **USB audio device**, so the PC sends sound to it like an ordinary speaker.
- Audio samples arrive in USB interrupts, are buffered, and are played out through the DAC to the on-board speaker.
- The joystick connects or disconnects playback.
- The **potentiometer** sets the volume. It is read with the ADC and scaled before the samples go to the DAC.

## Games

- **Snake** for sure, plus **Pong** or **Tic-Tac-Toe** if time allows.
- Controlled with the joystick. The game updates on a fixed tick from an RTX virtual timer, so speed does not depend on how busy the other threads are.

## Why this design is interesting

The three apps stress the RTOS in different ways:

- **Audio** has a hard real-time deadline. A late buffer is an audible click.
- **The gallery** moves a lot of data to the LCD but has no deadline.
- **The games** need steady timing and quick response to input.

Running them under one scheduler, with a shared LCD, is where the labs on priorities, mutexes and timers come together.
