# Completing Streams

## In Simple Terms

"Completing a stream" means the publisher has finished sending all its data
successfully and calls `onComplete()` exactly once. After that, the subscriber knows
for certain no more items are coming — it's a clean, successful end to the sequence.

## Simple Example

```java
Flux.just("a", "b", "c")
    .subscribe(
        item -> System.out.println("Item: " + item),
        error -> System.out.println("Error: " + error), // won't fire here
        () -> System.out.println("Stream completed successfully!")
    );
```

For an infinite stream, completion may never happen naturally (e.g.,
`Flux.interval(...)` never completes on its own) — you'd need an operator like
`.take(5)` to force completion after 5 items:

```java
Flux.interval(Duration.ofSeconds(1))
    .take(5) // forces completion after 5 items
    .subscribe(
        tick -> System.out.println("Tick: " + tick),
        error -> {},
        () -> System.out.println("Completed after 5 ticks!")
    );
```

## Why It Matters

Knowing exactly when (and if) a stream completes matters for cleanup logic and for
composing pipelines correctly — for example, `.collectList()` needs the source to
complete before it can hand back the full aggregated list; an infinite, never-
completing `Flux` would make `.collectList()` wait forever.
