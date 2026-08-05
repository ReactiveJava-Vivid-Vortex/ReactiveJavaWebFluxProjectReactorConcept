# Flux — Topic Overview

## What Is This Topic About? (In Simple Terms)

If `Mono` is "0 or 1 value," then `Flux<T>` is its bigger sibling: **0 to
potentially infinite values**, emitted over time. Everything you learned about
`Mono` (laziness, the three-outcome lifecycle, factory methods) applies here too —
just repeated for many items instead of one.

The main skill in this topic is picking the right way to **create** a `Flux` for
your data source:

- Already have the data? → `Flux.just()` / `Flux.fromIterable()`
- Need to generate values one at a time, synchronously? → `Flux.generate()`
- Need to bridge an external, asynchronous, push-based source (a listener, a
  message queue)? → `Flux.create()` / `Flux.push()`
- Need a repeating timer? → `Flux.interval()`

```java
Flux.range(1, 5)
    .map(n -> n * n)
    .subscribe(square -> System.out.println("Square: " + square));
// Square: 1, 4, 9, 16, 25
```

A second key idea is the distinction between **finite** streams (they eventually
call `onComplete()` on their own, like `Flux.range()`) and **infinite** streams
(they never complete unless you force them to, like `Flux.interval()`) — infinite
streams must always be bounded explicitly with something like `.take(n)`, or they'll
run forever.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---------|-------------------|
| 1 | **Flux.just()** | Emits a small, fixed, already-known set of values, then completes. |
| 2 | **Flux.range()** | Lazily emits a sequence of consecutive integers, respecting backpressure. |
| 3 | **Flux.fromIterable()** | Emits each element of an existing `List`/collection, one at a time. |
| 4 | **Flux.generate()** | Synchronously produces items one at a time, in direct response to demand — great for stateful sequences. |
| 5 | **Flux.create()** | Bridges external, async, push-based sources (listeners, queues) — supports multi-threaded emission. |
| 6 | **Flux.push()** | Like `create()`, but optimized for a single-threaded producer only. |
| 7 | **Flux.interval()** | Emits an incrementing counter on a fixed time interval, forever, until bounded (e.g., with `.take()`). |
| 8 | **Infinite streams** | Never call `onComplete()` on their own — must be explicitly bounded (`.take(n)`, `.take(Duration)`). |
| 9 | **Finite streams** | Eventually complete on their own once their known, bounded data is exhausted. |
| 10 | **FluxSink** | The manual emission "microphone" inside `Flux.create()`/`Flux.push()` — call `next()` many times, then `complete()`/`error()`. |
| 11 | **Custom publishers** | Wrapping a proprietary/legacy data source behind `Flux.create()`/`generate()` so the rest of your code treats it like any other stream. |
| 12 | **Event generation** | The broader pattern of producing application events as a `Flux` for others to react to (foundation for `Sinks`, covered later). |

## How It All Fits Together

```
Data source shape?
   │
   ├── Already in memory (List, fixed values) ──▶ Flux.just() / Flux.fromIterable()
   │
   ├── Generate synchronously, one at a time ───▶ Flux.generate()
   │
   ├── External, async, push-based source ──────▶ Flux.create() / Flux.push()
   │
   └── Repeating timer ──────────────────────────▶ Flux.interval()  (bound with .take()!)
```

Once you're comfortable creating a `Flux` the right way for any given source, the
next topic — Reactor Operators — is where you'll learn to transform, filter, and
reshape everything flowing through it.
