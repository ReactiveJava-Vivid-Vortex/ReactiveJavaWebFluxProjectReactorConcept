# Mono.defer()

## In Simple Terms

`Mono.defer(supplier)` lets you **postpone building the actual `Mono` itself** until
subscription time — the supplier you pass returns a whole new `Mono` for each
subscriber. This is different from `Mono.fromSupplier()`, which returns a *value*;
`Mono.defer()`'s supplier returns a whole *new Mono/pipeline*.

This is especially useful when the choice of *which* `Mono` to return depends on
state that should be evaluated fresh at subscription time (not at pipeline-build
time).

## Simple Example

```java
Mono<String> withoutDefer = Mono.just(getStatus()); // getStatus() called immediately

Mono<String> withDefer = Mono.defer(() -> Mono.just(getStatus())); // deferred, re-evaluated per subscription

// Simulate the value of getStatus() changing over time:
String status = "PENDING";
withDefer.subscribe(s -> System.out.println("First check: " + s));

status = "COMPLETED"; // (in real code, this would be some external mutable state)
withDefer.subscribe(s -> System.out.println("Second check: " + s));
```

A very common real use case: choosing between a cache hit and a fresh database call
at subscription time:

```java
Mono<User> getUser(String id) {
    return Mono.defer(() -> {
        User cached = cache.get(id);
        return cached != null ? Mono.just(cached) : userRepository.findById(id);
    });
}
```

## Why It Matters

`Mono.defer()` is essential whenever the *decision of what to do* must happen fresh,
per subscription, rather than once when the pipeline is assembled. Without it, values
or branching logic might get "baked in" too early, leading to stale or incorrect
behavior on repeated subscriptions.
