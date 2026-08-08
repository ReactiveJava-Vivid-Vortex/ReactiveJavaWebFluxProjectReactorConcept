# Success vs Empty vs Error

## In Simple Terms

Every `Mono` ends up in exactly one of three states — and it's worth treating
each one differently in your code instead of lumping them together:

| State   | Signal(s)                     | What It Means                                  |
|---------|--------------------------------|-------------------------------------------|
| Success | `onNext(value)` + `onComplete()` | It worked, here's the result   |
| Empty   | `onComplete()` only             | It worked, but there's nothing to give back (e.g. "not found") |
| Error   | `onError(throwable)`            | It failed                          |

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

A common beginner mistake is mixing up "empty" and "error" — like throwing an
exception every time something isn't found. Keeping them separate lets you use
the right tool for each case: `switchIfEmpty()` for a legitimate "nothing here,"
and `onErrorResume()`/`onErrorReturn()` for actual failures.
