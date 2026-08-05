# Mono.empty()

## In Simple Terms

`Mono.empty()` creates a `Mono` that **completes successfully without ever emitting
any value**. This represents "nothing to return, but not an error either" — like a
database lookup that legitimately found no matching record.

## Simple Example

```java
Mono<String> empty = Mono.empty();

empty.subscribe(
    value -> System.out.println("Value: " + value),   // never called
    error -> System.out.println("Error: " + error),    // never called
    () -> System.out.println("Completed with no value") // this fires
);
// Output: Completed with no value
```

A common real-world use: representing "user not found" without throwing an error:

```java
public Mono<User> findUser(String id) {
    if (!database.containsKey(id)) {
        return Mono.empty(); // no user found, but not an error
    }
    return Mono.just(database.get(id));
}
```

Callers can then handle the empty case explicitly with `.switchIfEmpty(...)`:

```java
findUser("123")
    .switchIfEmpty(Mono.error(new UserNotFoundException("123")))
    .subscribe(user -> System.out.println(user));
```

## Why It Matters

Distinguishing "empty" from "error" is important semantically — an empty result
usually isn't a failure, just an absence of data. Reactive code that conflates the
two (e.g., always throwing an exception instead of returning `Mono.empty()`) makes
composition and error handling unnecessarily awkward.
