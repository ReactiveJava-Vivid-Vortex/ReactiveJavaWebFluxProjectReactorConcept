# Course Overview

## In Simple Terms

This section is the map for the whole course. Before writing a single line of reactive
code, we need to understand **why** reactive programming exists at all. That means
stepping back from Java syntax and looking at things closer to the metal: how a CPU
runs your code, how the operating system juggles many programs at once, and what
"waiting" actually costs a program.

Once those fundamentals are clear, everything Project Reactor does — `Mono`, `Flux`,
schedulers, backpressure — will feel like a natural answer to a real problem, instead
of new syntax to memorize.

## The Roadmap

1. How a computer actually executes your code (processes, threads, CPU scheduling).
2. Why some operations "block" and why that is expensive.
3. Synchronous vs asynchronous programming.
4. Blocking vs non-blocking I/O, and the various I/O models.
5. Finally: why reactive programming was invented, and when it helps.

## Simple Example

Think of a restaurant:

- A **blocking/synchronous** waiter takes an order, walks it to the kitchen, and then
  just **stands there** waiting until the food is ready before serving the next table.
- A **non-blocking/reactive** waiter takes an order, hands it to the kitchen, and
  immediately goes to take the next table's order. When food becomes ready (an event),
  the waiter is notified and delivers it.

The second waiter serves far more tables with the same number of people. That single
idea — don't wait, get notified instead — is the seed of everything in this course.
