# CPU Scheduling

## In Simple Terms

A CPU core can only run **one instruction stream at a time**. But your computer runs
dozens of programs "at once" (browser, IDE, music player...). How? The operating
system rapidly switches the CPU between different threads, giving each one a small
slice of time (a few milliseconds). This is called **CPU scheduling**.

It feels simultaneous to us because the switching happens so fast, but at any given
instant, a single core is doing exactly one thing.

## Simple Example

Imagine a CPU core as a single cashier at a store, and threads as customers in line:

```
Time slice 1: Cashier serves Thread-A for 10ms
Time slice 2: Cashier serves Thread-B for 10ms
Time slice 3: Cashier serves Thread-C for 10ms
Time slice 4: Cashier serves Thread-A again for 10ms
...
```

The OS scheduler decides the order and duration of these slices, based on priority,
fairness, and whether a thread is waiting on something (like disk or network).

## Why It Matters for Reactive Programming

If a thread is **blocked** (e.g., waiting for a database response), it is still
"holding a seat" even though it's doing no useful work — the scheduler still has to
manage it, and no other work can use that thread meanwhile. Reactive programming tries
to make sure threads are **never idle-waiting**; they should always be scheduled to do
actual work, and be released back to the pool the instant they'd otherwise block.
