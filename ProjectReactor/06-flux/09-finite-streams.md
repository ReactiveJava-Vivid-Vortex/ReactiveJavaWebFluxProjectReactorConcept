# Finite Streams

## In Simple Terms

A **finite stream** is a `Flux` that eventually finishes on its own — it has a
known, limited number of items (even if that number is huge), unlike an infinite
stream that keeps going unless something stops it from outside.

## Simple Example

```java
Flux<Integer> finite = Flux.range(1, 1_000_000); // large, but still finite

finite.subscribe(
    n -> { /* process each */ },
    error -> System.out.println("Error: " + error),
    () -> System.out.println("All 1,000,000 items processed!") // WILL fire eventually
);
```

Compare that with `Flux.interval(...)`, which is **not** finite — it never calls
`onComplete()` on its own, no matter how long you wait.

```
Flux.range(1, 5)          -> finite  (completes after 5 items)
Flux.fromIterable(list)   -> finite  (completes after the list is exhausted)
Flux.interval(Duration)   -> infinite (never completes on its own)
```

## Why It Matters

Whether a stream is finite or infinite matters for operators that need the whole
stream to finish before they can give you anything — like `.collectList()`,
`.count()`, or `.reduce()`. Those would just sit and wait forever on a truly
infinite `Flux`, since they can't produce a result until `onComplete()` fires.
