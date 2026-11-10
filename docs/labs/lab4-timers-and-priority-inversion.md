# Lab 4: Real-time scheduling and priority inversion

**Goal:** use RTX virtual timers alongside a chain of signalling threads, then observe priority inversion.

## What I did

**Part 1:** three threads at high, normal and low priority pass control around a ring with signals (1 → 2 → 3 → 1). Each one lights its own LED while it runs. Two periodic **virtual timers** (software timers run by the RTX timer thread) flash two more LEDs at different periods. In the Event Viewer I could see the hand-offs and where `osTimerThread` ran.

**Part 2:** I ran the course's priority-inversion example. A low-priority thread holds a shared resource, a high-priority thread waits for it, and a medium-priority thread runs in between and delays the high-priority one. I compared that with **priority elevation**, where the resource holder temporarily runs at a higher priority so the high-priority thread is no longer stuck behind the medium one.

## Excerpt (my code)

```c
osTimerDef(timer0_handle, callback);
osTimerDef(timer1_handle, callback);

osTimerId timer_0 = osTimerCreate(osTimer(timer0_handle), osTimerPeriodic, (void *)0);
osTimerId timer_1 = osTimerCreate(osTimer(timer1_handle), osTimerPeriodic, (void *)1);
osTimerStart(timer_0, 40000);
osTimerStart(timer_1, 20000);

void led_Thread1 (void const *argument) {
    for (;;) {
        LED_On(0);
        delayC();                          // simulated work
        osSignalSet(T_led_ID2, 0x01);      // pass control to thread 2
        LED_Off(0);
        osSignalWait(0x03, osWaitForever); // wait for thread 3
    }
}
```

## What I learned

- Priority inversion happens whenever threads at different priorities share a resource. The fix is a protocol such as priority inheritance (which RTX mutexes provide) or priority ceiling, not just "give it a higher priority".
- All virtual timer callbacks run in a single RTX timer thread, one after another.

## What I would do differently

My timer callback calls `osDelay(1000)` to keep an LED on. That blocks the timer thread for a full second, so every other virtual timer is late during that time. A callback should be short: set a flag or signal a thread, and let the thread do the waiting.

## Used in the media center

- Game ticks (Snake, Pong) come from a periodic virtual timer whose callback only signals the game thread.
- The LCD is shared by every app, so it is protected by an RTX mutex, whose priority inheritance prevents the inversion seen in Part 2.
