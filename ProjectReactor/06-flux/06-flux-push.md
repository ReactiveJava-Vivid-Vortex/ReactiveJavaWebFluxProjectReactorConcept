# Flux.push()

## In Simple Terms

`Flux.push()` is basically `Flux.create()`'s sibling — it also gives you a
`FluxSink` to push items into — but it assumes **only one thread** will ever call
`sink.next()` at a time. Since it doesn't need to guard against multiple threads
stepping on each other, it can be a bit more efficient in that specific case.

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

Picking `push()` when you truly only have one producer thread avoids some
unnecessary internal overhead. If you're not sure whether multiple threads might
call the sink at once, play it safe and use `Flux.create()` instead — using
`push()` incorrectly with multiple threads leads to subtle, annoying bugs.
