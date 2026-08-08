# Subscription (Mono)

## In Simple Terms

Subscribing to a `Mono` is what actually kicks off its work and lets you react to
its result — a value, an empty finish, or an error. `.subscribe()` comes in a
few flavors depending on which of those you want to handle.

## Simple Example

```java
Mono<String> mono = Mono.just("Hello");

// 1. No handling at all (fire-and-forget, use with caution)
mono.subscribe();

// 2. Handle only the value
mono.subscribe(value -> System.out.println("Value: " + value));

// 3. Handle value + error
mono.subscribe(
    value -> System.out.println("Value: " + value),
    error -> System.out.println("Error: " + error)
);

// 4. Handle value + error + completion
mono.subscribe(
    value -> System.out.println("Value: " + value),
    error -> System.out.println("Error: " + error),
    () -> System.out.println("Completed")
);
```

Every `.subscribe()` call gives you back a `Disposable`, which lets you cancel it
by hand:

```java
Disposable subscription = mono.subscribe(v -> System.out.println(v));
subscription.dispose(); // cancels if still in progress
```

## Why It Matters

In a Spring WebFlux app, you basically never call `.subscribe()` yourself — the
framework does it for you, at the right time, and even handles cancellation
automatically if the client disconnects. Still, understanding how manual
subscribing works is essential for standalone code and tests, and for
understanding what the framework is quietly doing on your behalf.
