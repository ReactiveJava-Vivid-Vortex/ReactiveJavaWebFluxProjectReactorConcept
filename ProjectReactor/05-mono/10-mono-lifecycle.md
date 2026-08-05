# Mono Lifecycle

## In Simple Terms

A `Mono<T>` represents **0 or 1** asynchronous value. Its lifecycle follows the same
Reactive Streams signals as any publisher, but constrained to at most one item:

```
onSubscribe()
  -> onNext(value) [at most once]
  -> onComplete()  [always called, whether or not a value was emitted]

  OR

onSubscribe()
  -> onError(throwable) [instead of onNext + onComplete]
```

There are exactly **three possible outcomes** for any `Mono`:

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

Knowing there are exactly these three, mutually-exclusive outcomes makes writing
`StepVerifier` tests and reasoning about `Mono`-returning methods much simpler — you
always know a `Mono` will settle into exactly one of these three states, never a
mixture.

**This isn't a Mono-specific rule** — it's the universal Reactive Streams signal
grammar (`onSubscribe onNext* (onError | onComplete)?`) applied with one extra
constraint: `onNext` is capped at *at most once*. A `Flux` allows unlimited
`onNext` calls; a `Mono` allows 0 or 1. Same three signals, same terminal rules,
just a different cap. See [[the-three-signal-types]] for the full picture.
