# Operating System Scheduler

## In Simple Terms

The **OS scheduler** is the traffic cop deciding which thread gets to use which
CPU core, and for how long. It's making these decisions constantly, many times per
second.

When a thread asks for something it doesn't have yet — like data from a network
connection — the scheduler marks it as "waiting" and hands the CPU to a different
thread instead. Once the data shows up, the OS wakes that thread back up and gets
it back in line.

## Simple Example

Think of the scheduler as air traffic control at a busy airport:

```
Thread-A: ready to fly       -> gets a runway (CPU core)
Thread-B: waiting for cargo   -> parked at the gate, not using a runway
Thread-C: ready to fly       -> gets a runway
Thread-B: cargo has arrived!  -> back in line for a runway
```

No runway time gets wasted on Thread-B while it waits. But Thread-B is still
sitting there, parked, using up memory and being tracked by the airport.

## Why It Matters for Reactive Programming

Switching the CPU from one thread to another (a "context switch") isn't free —
it takes real time. If you have thousands of threads mostly just waiting around,
the OS spends a surprising chunk of its time simply juggling them. Reactive
programming shrinks the *number* of threads needed, so there's a lot less
juggling to do.
