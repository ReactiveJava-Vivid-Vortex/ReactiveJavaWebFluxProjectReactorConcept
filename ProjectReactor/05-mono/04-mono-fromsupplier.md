# Mono.fromSupplier()

## In Simple Terms

`Mono.fromSupplier(supplier)` creates a `Mono` that **lazily** calls a `Supplier<T>`
only when subscribed, and emits whatever value it returns. Unlike `Mono.just()`
(which captures the value immediately), the supplier's code doesn't run until
subscription happens — and it runs **fresh, every time**, for every new subscriber.

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

Notice "Computing value..." runs **twice** — once per subscriber — and each gets a
(slightly) different timestamp, because the supplier re-executes each time.

If the supplier returns `null`, the resulting `Mono` simply completes empty (like
`Mono.empty()`), instead of throwing.

## Why It Matters

`Mono.fromSupplier()` is the correct way to wrap a **synchronous, potentially
expensive** computation (e.g., reading a local cache, computing a hash) so it's
deferred until actually needed, rather than eagerly executed at pipeline-build time
like `Mono.just()` would.
