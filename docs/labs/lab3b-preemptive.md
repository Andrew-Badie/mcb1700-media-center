# Lab 3b: Pre-emptive scheduling with RTX

**Goal:** give threads different priorities, watch pre-emption happen, and compare it with round-robin.

## What I did

**Question 1:** five compute threads (A to E) at high, normal and low priority, each evaluating a different series, for example sums with powers and factorials. In the Event Viewer, the high-priority thread ran to completion first, then the normal ones, then the low ones. Threads at the same priority still shared time.

**Question 2:** the five-thread system from Lab 3a, re-run with the delays removed so that priority alone decided the order.

## Excerpt (my code)

```c
osThreadDef(ThreadC, osPriorityHigh,   1, 0);
osThreadDef(ThreadA, osPriorityNormal, 1, 0);
osThreadDef(ThreadD, osPriorityNormal, 1, 0);
osThreadDef(ThreadB, osPriorityLow,    1, 0);
osThreadDef(ThreadE, osPriorityLow,    1, 0);

void ThreadD (void const *argument) {
    for (;;) {
        valueD += pow(5, d) / factorial(d);   // series term
        d++;
        if (d == 5) osThreadTerminate(tid3_Thread);
    }
}
```

## What I learned

- Pre-emption gives fast response to urgent work. A higher-priority thread runs as soon as it is ready.
- The cost is starvation: a high-priority thread that never blocks keeps every lower thread from running.
- RTX combines both: higher priorities pre-empt, and equal priorities share time by round-robin.

## What I would do differently

`factorial()` returns a 32-bit `unsigned int`, which overflows after 12!. Thread B goes up to 15!, so its later terms were wrong. I would compute each term from the previous one (term *= x / n), which avoids both the overflow and recomputing the factorial every iteration.

## Used in the media center

The audio thread gets the highest priority so playback never stutters, and it must block between buffers so it cannot starve the UI.
