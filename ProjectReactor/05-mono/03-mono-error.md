# Mono.error()

## In Simple Terms

`Mono.error(throwable)` builds a `Mono` that, the moment someone subscribes,
immediately says "this failed" — no value is ever given out. It's basically
`throw new SomeException()`, but as a value you can return from a method instead
of literally throwing.

## Simple Example

```java
Mono<User> mono = Mono.error(new IllegalArgumentException("User ID cannot be null"));

mono.subscribe(
    user -> System.out.println("User: " + user),         // never called
    error -> System.out.println("Error: " + error.getMessage()) // fires
);
// Output: Error: User ID cannot be null
```

Common pattern — returning an error conditionally from a service method:

```java
public Mono<User> getUser(String id) {
    if (id == null) {
        return Mono.error(new IllegalArgumentException("id must not be null"));
    }
    return userRepository.findById(id);
}
```

**Watch out:** `Mono.error(new RuntimeException(...))` builds that exception
object right away — even before anyone subscribes. If building the exception is
expensive, use `Mono.error(() -> new RuntimeException(...))` (the supplier
version) so it's only built when actually needed.

## Why It Matters

Since `Mono.error()` is just a value, it fits naturally into the rest of your
pipeline — you can chain `onErrorResume()`, `onErrorReturn()`, or `retry()` on
top of it, without ever needing a regular try/catch block.
