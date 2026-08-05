# Signals Are Sequential, Never Concurrent

## In Simple Terms

The Reactive Streams specification guarantees that signals delivered to a single
`Subscriber` — `onSubscribe`, `onNext`, `onError`, `onComplete` — are always
delivered **one at a time, never concurrently**, even if the underlying work
happens across multiple threads. In other words: for any *one* subscription, you
will never see two `onNext()` calls overlapping in time.

This is a formal rule of the spec (Reactive Streams Rule 1.3), and it's a huge
relief once you know it: **you do not need to add your own synchronization
(`synchronized`, locks) inside a single subscriber's callback methods**, purely to
protect against concurrent signal delivery — the framework already guarantees that
won't happen.

## Simple Example

```java
Flux.range(1, 1000)
    .parallel(4)                  // work happens on 4 threads...
    .runOn(Schedulers.parallel())
    .map(n -> n * n)
    .sequential()                 // ...but re-joined into ONE sequential stream here
    .subscribe(n -> {
        // This lambda is called ONE AT A TIME, even though the upstream
        // work was parallelized across 4 threads — no synchronization needed here.
        System.out.println("Got: " + n);
    });
```

Even though `.parallel(4)` genuinely uses multiple threads to *compute* results,
Reactor guarantees the final `.subscribe()` callback still receives each `onNext()`
one at a time, never two calls racing each other.

## Why It Matters

Without this guarantee, every subscriber would need defensive locking around any
shared/mutable state touched inside its callbacks — a huge, error-prone burden.
Because Reactive Streams guarantees sequential delivery per subscription, you can
safely maintain simple, non-thread-safe local state (like a counter or running
total) inside a subscriber's callback, without extra synchronization — **as long as
you have exactly one subscription** doing the mutating. (Multiple *different*
subscribers/subscriptions can still run concurrently with each other — this
guarantee is per-subscription, not global.)
