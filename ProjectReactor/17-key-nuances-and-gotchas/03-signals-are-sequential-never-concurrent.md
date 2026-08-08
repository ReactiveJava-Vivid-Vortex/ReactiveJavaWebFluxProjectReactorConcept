# Signals Are Sequential, Never Concurrent

## In Simple Terms

The Reactive Streams rules guarantee that signals sent to any *one*
subscriber — `onSubscribe`, `onNext`, `onError`, `onComplete` — always
arrive one at a time, never overlapping, even if the actual work behind
them is happening across several threads. In other words: for any single
subscription, you'll never see two `onNext()` calls fighting for the same
moment in time.

This is an actual rule in the spec, and it's a huge relief once you know
about it: **you don't need to add your own locking inside a single
subscriber's callback methods** just to guard against concurrent signal
delivery — the framework already promises that won't happen.

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

Even though `.parallel(4)` really does use multiple threads to compute
results, Reactor guarantees the final `.subscribe()` callback still gets
each `onNext()` one at a time, never two calls racing each other.

## Why It Matters

Without this promise, every subscriber would need defensive locking around
any shared state touched inside its callbacks — a huge, error-prone chore.
Because signal delivery is guaranteed sequential per subscription, you can
safely keep simple, non-thread-safe local state (a counter, a running
total) inside a subscriber's callback without any extra locking — as long
as there's exactly one subscription doing the mutating. (Different
subscribers can still run concurrently *with each other* — this guarantee
is per-subscription, not global.)
