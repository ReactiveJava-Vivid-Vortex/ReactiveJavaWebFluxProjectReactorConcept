# Subscription (Mono)

## In Simple Terms

Subscribing to a `Mono` is what actually **triggers** its execution and lets you
react to its single value, empty completion, or error. `Mono.subscribe()` has several
overloads depending on which signals you care about.

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

Every `.subscribe()` call returns a `Disposable`, which lets you cancel the
subscription manually:

```java
Disposable subscription = mono.subscribe(v -> System.out.println(v));
subscription.dispose(); // cancels if still in progress
```

## Why It Matters

In a Spring WebFlux application, you typically **never call `.subscribe()` yourself**
— the framework subscribes to the `Mono`/`Flux` your controller returns, at the right
time, and manages cancellation automatically if the client disconnects. Understanding
manual subscription is still essential for writing standalone code, tests, and
understanding what the framework does on your behalf.
