# Reactive Streams Specification — Topic Overview

## What Is This Topic About? (In Simple Terms)

Before Project Reactor existed, a group of engineers (from Netflix, Lightbend,
Pivotal, and others) agreed on a common **rulebook** for how asynchronous,
non-blocking data streams should behave — so different libraries could interoperate.
That rulebook is the **Reactive Streams specification**, and it defines just four
tiny interfaces: `Publisher`, `Subscriber`, `Subscription`, and `Processor`.

Think of it like a contract between a **broadcaster** (`Publisher`, someone who has
data to send over time) and a **viewer** (`Subscriber`, someone who wants to receive
it). But unlike a real TV broadcast, the viewer isn't forced to watch faster than
they can process — the viewer holds a **remote control** (`Subscription`) and can
say "send me exactly 5 more items" (`request(5)`) or "stop, I'm done"
(`cancel()`). This consumer-controlled pacing is called **backpressure**, and it's
the single most important idea in the whole specification — it's what prevents a
fast producer from drowning a slow consumer in more data than it can handle.

Every stream follows a strict lifecycle: `onSubscribe()` sets things up, then zero
or more `onNext()` calls deliver items, and finally exactly one of `onComplete()`
(success) or `onError()` (failure) ends it — never both, and nothing after.

**Memorize this above everything else in this topic:** the entire data-flow
vocabulary of Reactive Streams is exactly **three signal types** —
`onNext` (repeatable), `onComplete` (terminal, success), `onError` (terminal,
failure) — plus the one-time `onSubscribe` handshake that isn't itself a data
signal. Every operator you'll ever use is just code reacting to one of these three.
See [[the-three-signal-types]] for the full breakdown.

```java
Publisher<Integer> publisher = subscriber -> {
    subscriber.onSubscribe(new Subscription() {
        public void request(long n) {
            subscriber.onNext(42);
            subscriber.onComplete();
        }
        public void cancel() {}
    });
};
```

Project Reactor's `Mono` and `Flux` are simply well-built, production-ready
implementations of this exact `Publisher` contract — so understanding these four
interfaces here means understanding the foundation everything else in this course is
built on.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---------|-------------------|
| 1 | **Publisher** | Produces a stream of data over time; does nothing until a `Subscriber` calls `subscribe()`. |
| 2 | **Subscriber** | Consumes data from a `Publisher`, reacting to `onSubscribe`, `onNext`, `onError`, `onComplete`. |
| 3 | **Subscription** | The "remote control" a subscriber uses to `request(n)` more items or `cancel()` the stream. |
| 4 | **Processor** | Both a `Subscriber` and a `Publisher` at once — a middle-of-the-pipeline bridge (mostly replaced by `Sinks` today). |
| 5 | **request(n)** | The subscriber's way of saying "send me exactly n more items" — the core mechanism of backpressure. |
| 6 | **cancel()** | The subscriber's way of saying "stop sending me data, I'm done" — releases resources early. |
| 7 | **Backpressure** | The overall mechanism letting a slow consumer control a fast producer's pace, preventing memory overload. |
| 8 | **Stream lifecycle** | `onSubscribe` → zero-or-more `onNext` → exactly one of `onComplete` or `onError`, never both, nothing after. |
| 9 | **Demand management** | The running balance of "items requested but not yet delivered," tracked to keep producer and consumer in sync. |
| 10 | **onNext()** | Signal for a single new item; called 0+ times for a `Flux`, at most once for a `Mono`. |
| 11 | **onComplete()** | Terminal "success, no more data coming" signal; called at most once, only on success. |
| 12 | **onError()** | Terminal "something went wrong" signal; called at most once, ends the stream immediately. |
| 13 | **The Three Signal Types** | The whole vocabulary in one rule: `onSubscribe onNext* (onError \| onComplete)?` — memorize this grammar. |

## How It All Fits Together

```
Subscriber.subscribe(Publisher)
        │
        ▼
Publisher hands over a Subscription  →  onSubscribe(subscription)
        │
        ▼
Subscriber calls subscription.request(n)   (demand flows UPSTREAM)
        │
        ▼
Publisher sends onNext(item) × up to n     (data flows DOWNSTREAM)
        │
        ▼
Publisher sends exactly one terminal signal: onComplete() OR onError()
```

This four-interface contract — with backpressure baked in from the start — is what
`Mono` and `Flux` build on top of. Every operator you'll learn later (`map`,
`filter`, `flatMap`...) is really just a smarter `Publisher`/`Subscriber` pair
obeying these same rules underneath.
