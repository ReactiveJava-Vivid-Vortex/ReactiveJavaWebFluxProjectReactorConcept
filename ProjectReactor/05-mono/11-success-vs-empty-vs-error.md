# Success vs Empty vs Error

## In Simple Terms

Every `Mono` settles into exactly one of three states, and it's important to treat
each one differently in your code:

| State   | Signal(s)                     | Meaning                                  |
|---------|--------------------------------|-------------------------------------------|
| Success | `onNext(value)` + `onComplete()` | Operation succeeded, here's the result   |
| Empty   | `onComplete()` only             | Operation succeeded, but there's no result (e.g. "not found") |
| Error   | `onError(throwable)`            | Operation failed                          |

## Simple Example

```java
public Mono<User> findUser(String id) {
    if (id == null) {
        return Mono.error(new IllegalArgumentException("id required")); // ERROR
    }
    User user = database.get(id);
    if (user == null) {
        return Mono.empty(); // EMPTY: valid lookup, nothing found
    }
    return Mono.just(user); // SUCCESS
}
```

Handling all three explicitly:

```java
findUser(userId)
    .map(user -> "Found: " + user.getName())      // handles SUCCESS
    .switchIfEmpty(Mono.just("No user found"))    // handles EMPTY
    .onErrorResume(e -> Mono.just("Error: " + e.getMessage())) // handles ERROR
    .subscribe(System.out::println);
```

## Why It Matters

A common beginner mistake is to conflate "empty" with "error" (e.g., always throwing
an exception for "not found" cases). Distinguishing them clearly lets callers use the
right tool for each case — `switchIfEmpty()` for legitimate absence of data, and
`onErrorResume()`/`onErrorReturn()` for actual failures — which keeps error handling
logic focused and readable.
