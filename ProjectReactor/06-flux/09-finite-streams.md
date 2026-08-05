# Finite Streams

## In Simple Terms

A **finite stream** is a `Flux` that eventually calls `onComplete()` on its own —
it has a known, bounded number of items (even if that number is very large), unlike
an infinite stream that runs forever unless externally stopped.

## Simple Example

```java
Flux<Integer> finite = Flux.range(1, 1_000_000); // large, but still finite

finite.subscribe(
    n -> { /* process each */ },
    error -> System.out.println("Error: " + error),
    () -> System.out.println("All 1,000,000 items processed!") // WILL fire eventually
);
```

Contrast with `Flux.interval(...)`, which is **not** finite — it never calls
`onComplete()` by itself, no matter how long you wait.

```
Flux.range(1, 5)          -> finite  (completes after 5 items)
Flux.fromIterable(list)   -> finite  (completes after the list is exhausted)
Flux.interval(Duration)   -> infinite (never completes on its own)
```

## Why It Matters

Whether a stream is finite or infinite matters for operators that need to know "the
whole stream is done" — like `.collectList()`, `.count()`, or `.reduce()`. These
operators would simply never emit a result on a truly infinite stream, because they
have to wait for `onComplete()` before producing their aggregated output.
