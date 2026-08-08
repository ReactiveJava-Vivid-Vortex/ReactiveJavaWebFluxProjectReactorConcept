# CPU Scheduling

## In Simple Terms

A CPU core can only do **one thing at a time**. Yet your laptop somehow runs a
browser, an IDE, and music player all "at once." How? The operating system
constantly flips the CPU between different threads, giving each one a tiny sliver
of time — just a few milliseconds.

It *feels* like everything runs together because the switching is so fast. But at
any single instant, one CPU core is only ever doing one thing.

## Simple Example

Think of a CPU core as one cashier, and threads as customers in line:

```
Time slice 1: Cashier serves Thread-A for 10ms
Time slice 2: Cashier serves Thread-B for 10ms
Time slice 3: Cashier serves Thread-C for 10ms
Time slice 4: Cashier serves Thread-A again for 10ms
...
```

The operating system decides the order and how long each slice lasts — based on
priority, fairness, and whether a thread is just sitting there waiting on
something (like a disk or the network).

## Why It Matters for Reactive Programming

If a thread is stuck waiting (say, for a database to answer), it's still "holding
a spot in line" — even though it's doing nothing. The scheduler still has to track
it, and nobody else can use it in the meantime. Reactive programming tries to make
sure a thread is never just standing there waiting — it should always be doing
real work, or freed up immediately to help someone else.
