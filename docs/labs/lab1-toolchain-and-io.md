# Lab 1: Keil µVision and the Cortex-M3

**Goal:** set up the toolchain, bring up the MCB1700, and read the joystick.

## What I did

- Built and flashed a starter project in Keil µVision on the MCB1700 (LPC1768).
- Read the 5-way joystick and lit a different LED for each direction, with the direction also shown on the LCD.
- Used a 10 ms SysTick interrupt as a time base.
- Sent debug output through the Cortex-M3 **ITM** (Instrumentation Trace Macrocell) stimulus port, so `printf` works over the debug probe with no UART.

## Excerpt (my code)

Joystick direction to LED, only when the state changes:

```c
res = get_button();
if (res != 0 && res != last_state) {
    last_state = res;
    if      (last_state == 1)  { strcpy(text, "Select"); LED_Out(1);  }
    else if (last_state == 8)  { strcpy(text, "Up");     LED_Out(2);  }
    else if (last_state == 16) { strcpy(text, "Right");  LED_Out(4);  }
    else if (last_state == 32) { strcpy(text, "Down");   LED_Out(8);  }
    else if (last_state == 64) { strcpy(text, "Left");   LED_Out(16); }
}
```

## What I would do differently

- Replace the magic numbers with named constants (`JOY_SELECT`, `JOY_UP`, ...) and use a `switch`.
- Debounce the joystick in the SysTick handler instead of acting on every raw read.

## Used in the media center

The menu, gallery and games all read the joystick this way, and ITM `printf` is the main debug channel.
