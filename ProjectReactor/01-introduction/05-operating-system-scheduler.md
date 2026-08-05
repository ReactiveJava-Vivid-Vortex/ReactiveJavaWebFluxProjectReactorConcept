# Operating System Scheduler

## In Simple Terms

The **OS scheduler** is the part of the operating system that decides which thread
gets to run on which CPU core, and for how long. It runs constantly in the
background, making decisions many times per second.

When a thread calls something blocking — like reading from a network socket that has
no data yet — the OS scheduler marks that thread as "waiting" (not runnable) and gives
the CPU core to a different thread instead. When the data finally arrives, the OS
wakes the waiting thread back up and schedules it to run again.

## Simple Example

Think of the scheduler as air traffic control at a busy airport:

```
Thread-A: ready to run       -> scheduler assigns it a runway (CPU core)
Thread-B: waiting for I/O    -> parked at the gate, not using the runway
Thread-C: ready to run       -> scheduler assigns it a runway
Thread-B: I/O finished!      -> back in the queue for a runway
```

No runway (CPU) time is wasted on Thread-B while it waits — but Thread-B itself is
still "parked," consuming memory and being tracked by the OS.

## Why It Matters for Reactive Programming

Switching a thread from "waiting" to "running" (called a **context switch**) has a
real cost. If you have thousands of threads mostly blocked on I/O, the OS spends a
surprising amount of time just switching between them. Reactive programming reduces
the *number* of threads needed, so the OS scheduler has far less work to do overall.
