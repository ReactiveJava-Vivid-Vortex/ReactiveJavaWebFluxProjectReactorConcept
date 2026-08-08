# Mono Lifecycle

## In Simple Terms

A `Mono<T>` represents **0 or 1** value that will arrive eventually. It follows
the same signal rules as any publisher, just with a tighter limit — at most one
item, ever:

```
onSubscribe()
  -> onNext(value) [at most once]
  -> onComplete()  [always called, whether or not a value was emitted]

  OR

onSubscribe()
  -> onError(throwable) [instead of onNext + onComplete]
```

There are exactly **three possible outcomes** for any `Mono` — nothing else can
happen:

1. **Success with a value**: `onNext(value)` then `onComplete()`.
2. **Success with no value (empty)**: just `onComplete()`, no `onNext()`.
3. **Failure**: `onError(throwable)`, no `onNext()` or `onComplete()`.

## Simple Example

```java
// Outcome 1: value
Mono.just("hello").subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e),
    () -> System.out.println("Complete")
);
// Value: hello
// Complete

// Outcome 2: empty
Mono.empty().subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e),
    () -> System.out.println("Complete")
);
// Complete   (no "Value:" line!)

// Outcome 3: error
Mono.error(new RuntimeException("failed")).subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e),
    () -> System.out.println("Complete")
);
// Error: failed   (no "Complete" line!)
```

## Why It Matters

Knowing there are exactly these three outcomes — and never a mix of them —
makes writing `StepVerifier` tests and reasoning about `Mono`-returning methods
much easier.

**This isn't just a Mono thing** — it's the same universal rule every reactive
stream follows, just with `onNext` capped at one instead of unlimited. See
[[the-three-signal-types]] for the full picture.
