# Mono.defer()

## In Simple Terms

`Mono.defer(supplier)` puts off building the `Mono` itself until someone actually
subscribes — the code you pass builds a brand-new `Mono` for each subscriber.
This is different from `Mono.fromSupplier()`, which returns a *value*.
`Mono.defer()`'s code returns a whole *new Mono*, chosen fresh every time.

This matters most when deciding *which* `Mono` to return depends on something
that should be checked fresh, right when someone subscribes — not once when you
wrote the code.

## Simple Example

```java
Mono<String> withoutDefer = Mono.just(getStatus()); // getStatus() called immediately

Mono<String> withDefer = Mono.defer(() -> Mono.just(getStatus())); // re-checked per subscription

// Imagine getStatus() changes over time:
String status = "PENDING";
withDefer.subscribe(s -> System.out.println("First check: " + s));

status = "COMPLETED"; // (in real code, this would be some external mutable state)
withDefer.subscribe(s -> System.out.println("Second check: " + s));
```

A very common real use — deciding between a cache hit and a fresh database call,
checked fresh at subscription time:

```java
Mono<User> getUser(String id) {
    return Mono.defer(() -> {
        User cached = cache.get(id);
        return cached != null ? Mono.just(cached) : userRepository.findById(id);
    });
}
```

## Why It Matters

`Mono.defer()` matters whenever the *decision* itself needs to happen fresh, each
time someone subscribes — otherwise a value or a branch might get "locked in" too
early, giving you stale or wrong results on repeated subscriptions.
