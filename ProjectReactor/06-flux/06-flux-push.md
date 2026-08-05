# Flux.push()

## In Simple Terms

`Flux.push()` is very similar to `Flux.create()` — it gives you a `FluxSink` to
manually emit items — but it's designed for the case where **only a single producer
thread** will ever call `sink.next()` at a time (not multiple concurrent threads).
Because it doesn't need to handle multi-threaded producer synchronization, it can be
slightly more efficient than `Flux.create()` in that specific scenario.

## Simple Example

```java
Flux<Integer> flux = Flux.push(sink -> {
    // Imagine a single background thread feeding this sink
    Thread producer = new Thread(() -> {
        for (int i = 0; i < 5; i++) {
            sink.next(i);
        }
        sink.complete();
    });
    producer.start();
});

flux.subscribe(value -> System.out.println("Got: " + value));
```

## When to Use `push()` vs `create()`

| Scenario                                             | Use            |
|-------------------------------------------------------|----------------|
| Single thread emits events (e.g., one listener thread) | `Flux.push()`  |
| Multiple threads might emit events concurrently        | `Flux.create()` |

## Why It Matters

Choosing `push()` over `create()` when you genuinely have a single-threaded producer
avoids unnecessary internal synchronization overhead. If you're unsure whether
multiple threads could call the sink concurrently, it's safer to default to
`Flux.create()`, since using `push()` incorrectly with multiple threads can lead to
subtle, hard-to-reproduce bugs.
