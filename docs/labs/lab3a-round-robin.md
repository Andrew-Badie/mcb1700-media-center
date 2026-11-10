# Lab 3a: RTX multitasking with round-robin scheduling

**Goal:** run several threads at equal priority under RTX round-robin time slicing, then coordinate them with signals and a mutex.

## What I did

**Part 1:** three threads at the same priority, each doing its own work until a condition ends it: one counts up to 5000, one counts down to 0, one appends to a log string. I watched RTX give each thread a time slice in turn in the **Event Viewer**.

**Part 2:** five threads modelling parts of a system (memory access, CPU work, two loggers, a user task):

- Thread 1 sets an ADC control bit through its bit-band alias, then waits for a **signal**.
- Thread 2 runs a loop written to use conditional execution and the barrel shifter, then **signals** Thread 1.
- Threads 3 and 4 share a log buffer, protected by a **mutex**.
- Thread 5 runs at idle priority.

I followed the shared counters in a Watch window and the hand-offs in the Event Viewer.

## Excerpt (my code)

Signal hand-off between two threads:

```c
void Thread2 (void const *argument) {
    /* ... CPU-bound loop ... */
    osSignalSet(tid_Thread, 0x01);       // wake Thread 1
    osThreadTerminate(tid2_Thread);
}

void Thread1 (void const *argument) {
    memory_count++;
    ADCR_Bit28 = 1;                      // bit-band write from Lab 2
    evt = osSignalWait(0x01, 100);       // block until Thread 2 signals (100 ms timeout)
    osThreadTerminate(tid_Thread);
}
```

## What I learned

- Round-robin is fair and simple when tasks matter equally, but a thread's response time depends on how many others are ready. It is a poor fit for anything with a deadline.
- A signal says that something happened. A mutex protects data that more than one thread uses.

## What I would do differently

Reviewing the code now, I found two bugs:

- **Part 1:** the log thread stops when `sizeof(logger) == 999`. `sizeof` is fixed at compile time (1000 here), so that test is never true, and `strcat` keeps writing past the end of the buffer. The fix is to track the length and stop before the buffer is full, for example with `strlen()` and a bounded copy.
- **Part 2:** Thread 3 takes the mutex a second time and terminates without releasing it, so the mutex stays locked. Every `osMutexWait` needs a matching `osMutexRelease` before the thread exits.

## Used in the media center

The media center is split into threads (input, display, audio, game) that share data, and this lab is where I learned to coordinate them.
