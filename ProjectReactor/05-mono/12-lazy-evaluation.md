# Lazy Evaluation (Mono)

## In Simple Terms

Most `Mono` factory methods (`fromSupplier`, `fromCallable`, `defer`) don't
actually run anything until someone subscribes. Because of this, you can build up
complicated `Mono` chains and pass them around freely, without accidentally
triggering real work (like a network call) before it's actually needed.

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

Compare with `Mono.just(fetchData())`, where `fetchData()` runs **immediately**
the moment that line runs — no matter whether anyone ever subscribes.

## Why It Matters

Laziness means a `Mono` can be safely stored, passed around, or reused as a
"recipe" without accidentally triggering side effects too early or too many
times. It's also exactly why retries (`retry()`/`retryWhen()`) work correctly —
each retry subscribes again, which re-runs the lazy work from scratch.
