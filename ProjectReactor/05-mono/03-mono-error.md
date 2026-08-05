# Mono.error()

## In Simple Terms

`Mono.error(throwable)` creates a `Mono` that, once subscribed, immediately signals
failure via `onError(throwable)` — no value is ever emitted. It's the reactive
equivalent of `throw new SomeException()`, but wrapped as a value you can return from
a method instead of literally throwing.

## Simple Example

```java
Mono<User> mono = Mono.error(new IllegalArgumentException("User ID cannot be null"));

mono.subscribe(
    user -> System.out.println("User: " + user),         // never called
    error -> System.out.println("Error: " + error.getMessage()) // fires
);
// Output: Error: User ID cannot be null
```

Common usage — returning an error conditionally from a service method:

```java
public Mono<User> getUser(String id) {
    if (id == null) {
        return Mono.error(new IllegalArgumentException("id must not be null"));
    }
    return userRepository.findById(id);
}
```

**Important gotcha:** `Mono.error(new RuntimeException(...))` still constructs the
exception object eagerly (even before subscription). If constructing the exception
is expensive, use `Mono.error(() -> new RuntimeException(...))` (the supplier
overload) so it's only built lazily, per subscriber.

## Why It Matters

Because `Mono.error()` is a *value* (not a thrown exception), it composes naturally
with the rest of the reactive pipeline — you can `.subscribe()` and route it through
`onErrorResume()`, `onErrorReturn()`, or `retry()`, without ever leaving the reactive
flow or needing a synchronous try/catch block.
