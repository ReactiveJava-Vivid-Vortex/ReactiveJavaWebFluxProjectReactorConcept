# Course Overview

## In Simple Terms

This section is the map for everything that follows. Before we touch any reactive
code, we need to answer one question: **why does reactive programming even exist?**

To answer that, we have to step back from Java for a moment and look at how a
computer actually works — how a CPU runs your code, how the operating system
juggles many programs at once, and what "waiting" really costs.

Once that clicks, everything else in this course — `Mono`, `Flux`, schedulers,
backpressure — stops feeling like new syntax to memorize. It starts feeling like
the obvious answer to a real, physical problem.

## The Roadmap

1. How a computer runs your code (processes, threads, CPU scheduling).
2. Why "waiting" is expensive.
3. Synchronous vs asynchronous code.
4. Blocking vs non-blocking I/O.
5. Why reactive programming was invented, and when it actually helps.

## Simple Example

Picture a restaurant with one waiter.

- A **blocking** waiter takes an order, walks it to the kitchen, and then just
  **stands there** waiting for the food — ignoring every other table until it's
  ready.
- A **non-blocking** waiter takes an order, drops it at the kitchen, and
  immediately goes to take the next table's order. When food is ready, the
  kitchen rings a bell and the waiter delivers it.

Same one waiter. The second one serves far more tables. That's the whole idea
behind this course: **don't wait around — get notified instead.**
