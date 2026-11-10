# Lab 2: Cortex-M3 features for performance

**Goal:** compare ways of setting and clearing a single register bit, and see what the compiler and the Cortex-M3 do with them.

## What I did

I set and cleared single register bits three ways, each selected with the joystick:

1. **Masking:** read-modify-write on the whole GPIO register (`|=`, `&= ~`) for P1.28 and P2.2.
2. **Bit-band, computed at run time:** a macro turns a register address and bit number into its bit-band alias address, used on the same GPIO bits.
3. **Bit-band, fixed alias address:** precomputed alias addresses written directly as constants.

On the Cortex-M3, each bit in the SRAM and peripheral regions has its own word-sized alias. One store to the alias changes exactly one bit, atomically, with no read-modify-write. That matters when an interrupt and a thread touch the same register.

I timed each method at `-O0` and `-O3`, and looked at the disassembly for **conditional execution** (`IT` blocks instead of branches) and the **barrel shifter** (a shift folded into an add, so a multiply-and-add becomes one instruction).

## Excerpt (my code)

```c
// Alias address = region base | 0x02000000 | (byte offset << 5) | (bit << 2)
#define ADDRESS(x)    (*((volatile unsigned long *)(x)))
#define BitBand(x, y) ADDRESS(((unsigned long)(x) & 0xF0000000) | 0x02000000 | \
                      (((unsigned long)(x) & 0x000FFFFF) << 5) | ((y) << 2))

// Masking
LPC_GPIO1->FIOPIN |=  (1 << 28);
LPC_GPIO1->FIOPIN &= ~(1 << 28);

// Bit-band through the macro
bit1  = &BitBand(&LPC_GPIO1->FIOPIN, 28);
*bit1 = 1;
*bit1 = 0;
```

## Results

| Method | `-O0` | `-O3` | Change |
| --- | --- | --- | --- |
| Masking | 257.28 µs | 255.97 µs | 0.50 % |
| Bit-band (macro) | 836.39 µs | 835.08 µs | 0.15 % |
| Bit-band (fixed alias) | 547.63 µs | 546.32 µs | 0.12 % |

## What I would do differently

The optimisation level barely changed these numbers, and that is the real lesson. Each timed block also included `printf` calls over ITM and waits on the SysTick flag, so I mostly measured I/O and waiting, not the bit access itself. The modes were not even doing the same amount of work: the macro mode toggled the bits an extra time and waited on the SysTick flag twice, which is most of why it took longest. And the fixed-alias mode wrote to different register bits than the other two.

Next time I will:

- Make every mode do identical work on the same register bits.
- Time only the bit access, using the Cortex-M3 **DWT cycle counter** (`DWT->CYCCNT`) before and after, with no `printf` inside the timed region.
- Repeat each measurement many times and report the spread.
- Compare the instruction count for each method in the disassembly. A single bit-band store should beat a load-modify-store.

## Used in the media center

Fast, atomic bit access for GPIO that both an interrupt handler and a thread touch, and the habit of checking the disassembly before trusting a timing claim.
