# Publisher & Subscriber Implementation — Topic Overview

## What Is This Topic About? (In Simple Terms)

This topic is a deliberate step backward from convenience: instead of using
Reactor's ready-made `Mono`/`Flux`, you build a `Publisher` and `Subscriber`
completely **by hand**, using only the raw Reactive Streams interfaces from the
previous topic. It's like taking apart an engine to see every gear before driving
the finished car.

Why bother? Because doing this once makes you deeply appreciate what Reactor is
doing *for* you automatically: tracking outstanding demand, safely handling
`cancel()`, emitting `onComplete()` at exactly the right moment, and doing it all
correctly even under concurrent access. Writing your own `Subscription` — the object
responsible for counting how many items have been requested and stopping exactly
on cue — reveals just how much careful bookkeeping is involved.

```java
class SimpleSubscription implements Subscription {
    int index = 0;
    public void request(long n) {
        for (long i = 0; i < n && index < data.length; i++) {
            subscriber.onNext(data[index++]);
        }
        if (index == data.length) subscriber.onComplete();
    }
    public void cancel() { /* stop emitting */ }
}
```

The key idea threading through every subtopic here is **demand-driven publishing**:
a well-behaved publisher only ever produces an item in direct response to
`request(n)` — never eagerly, never more than asked. That discipline is exactly what
makes reactive streams memory-safe even with huge or infinite data sources.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---------|-------------------|
| 1 | **Implementing a custom Publisher** | Hand-write `subscribe()` to hand the subscriber a `Subscription` that emits items only on `request(n)`. |
| 2 | **Implementing a custom Subscriber** | Hand-write all four callbacks (`onSubscribe`, `onNext`, `onError`, `onComplete`), controlling your own request pace. |
| 3 | **Subscription (custom)** | The trickiest part: safely track outstanding demand and stop cleanly on `cancel()`, even under concurrency. |
| 4 | **Requesting elements** | Calling `request(n)` in controlled batches (not just `Long.MAX_VALUE`) to pace consumption deliberately. |
| 5 | **Cancelling subscriptions** | Calling `cancel()` (or `.dispose()` on the higher-level API) to stop emissions and free resources early. |
| 6 | **Completing streams** | A publisher calls `onComplete()` exactly once, only after successfully emitting everything it has. |
| 7 | **Error signaling** | A publisher calls `onError(throwable)` exactly once to end the stream on failure — no further signals follow. |
| 8 | **Demand-driven publishing** | The publisher only ever produces the next item in direct response to demand — never gets ahead of the consumer. |

## How It All Fits Together

```
Custom Publisher.subscribe(customSubscriber)
        │
        ▼
Hand out a hand-rolled Subscription
        │
        ▼
Subscriber calls request(n)  ──▶  Publisher emits AT MOST n onNext() calls
        │
        ▼
Publisher ends with exactly one: onComplete()  OR  onError()
        │
   (subscriber may also cancel() early at any point)
```

Everything you build here by hand is exactly what `Flux.range()`, `Flux.create()`,
and Reactor's internal machinery do for you — correctly, safely, and without you
needing to think about it — in every subsequent topic in this course.
