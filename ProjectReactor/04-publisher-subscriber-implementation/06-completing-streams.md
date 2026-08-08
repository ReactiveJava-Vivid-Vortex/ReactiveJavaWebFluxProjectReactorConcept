# Completing Streams

## In Simple Terms

A stream "completes" when the publisher has successfully sent everything it has
and calls `onComplete()` exactly once. After that, the subscriber knows for sure
nothing else is coming — it's a clean, successful ending.

## Simple Example

```java
Flux.just("a", "b", "c")
    .subscribe(
        item -> System.out.println("Item: " + item),
        error -> System.out.println("Error: " + error), // won't fire here
        () -> System.out.println("Stream completed successfully!")
    );
```

An endless stream might never complete on its own — `Flux.interval(...)`, for
example, never does. You'd need something like `.take(5)` to force it to stop
after 5 items:

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

Knowing when (and if) a stream actually completes matters for cleanup logic and
for chaining operators correctly. For example, `.collectList()` needs the stream
to finish before it can hand you the full list — an endless `Flux` would leave
`.collectList()` waiting forever.
