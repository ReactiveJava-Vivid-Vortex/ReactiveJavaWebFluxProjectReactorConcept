# Mono.fromSupplier()

## In Simple Terms

`Mono.fromSupplier(supplier)` runs a piece of code **only when someone
subscribes**, and hands out whatever that code returns. Unlike `Mono.just()`
(which grabs the value right away), the supplier doesn't run until subscription
— and it runs fresh, every single time, for every new subscriber.

## Simple Example

```java
Mono<String> mono = Mono.fromSupplier(() -> {
    System.out.println("Computing value...");
    return "Computed at " + System.currentTimeMillis();
});

System.out.println("Mono created, nothing has run yet");

mono.subscribe(value -> System.out.println("Subscriber 1: " + value));
mono.subscribe(value -> System.out.println("Subscriber 2: " + value));
```

Output:
```
Mono created, nothing has run yet
Computing value...
Subscriber 1: Computed at 1732000000000
Computing value...
Subscriber 2: Computed at 1732000000123
```

Notice "Computing value..." runs **twice** — once per subscriber — and each gets
a slightly different timestamp, because the code inside re-runs each time.

If the supplier returns `null`, the `Mono` just completes empty instead of
blowing up.

## Why It Matters

Use `Mono.fromSupplier()` whenever you have a synchronous, possibly expensive
piece of work (reading a local cache, computing a hash) that you want deferred
until it's actually needed — instead of running eagerly like `Mono.just()` would.
