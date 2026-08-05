# Lazy Evaluation (Mono)

## In Simple Terms

Most `Mono` factory methods (`fromSupplier`, `fromCallable`, `defer`) don't run their
logic until subscription happens. This "laziness" means you can safely build up
complex `Mono` chains as reusable blueprints, without triggering any real work (like
network calls) until something actually subscribes to them.

## Simple Example

```java
Mono<String> lazyMono = Mono.fromSupplier(() -> {
    System.out.println("Fetching data from remote service...");
    return "Remote Data";
});

System.out.println("Mono defined. No network call yet.");

// Nothing has happened until this line:
lazyMono.subscribe(data -> System.out.println("Received: " + data));
```

Output:
```
Mono defined. No network call yet.
Fetching data from remote service...
Received: Remote Data
```

Contrast this with `Mono.just(fetchData())`, where `fetchData()` would run
**immediately**, the instant that line executes — regardless of whether anyone ever
subscribes.

## Why It Matters

Lazy evaluation means a `Mono` can be safely passed around, stored, or reused as a
"recipe" for an operation, without accidentally triggering side effects too early or
too often. It also enables retry logic (`retry()`/`retryWhen()`) to work correctly —
each retry re-subscribes, which re-triggers the lazy computation fresh.
