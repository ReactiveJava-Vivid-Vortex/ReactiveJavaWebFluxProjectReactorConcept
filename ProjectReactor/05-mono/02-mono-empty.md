# Mono.empty()

## In Simple Terms

`Mono.empty()` finishes successfully **without ever handing out a value**. Think
of it as "nothing to give you, but nothing went wrong either" — like searching a
database and legitimately finding no match.

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

A common real use — saying "user not found" without treating it as an error:

```java
public Mono<User> findUser(String id) {
    if (!database.containsKey(id)) {
        return Mono.empty(); // no user found, but not an error
    }
    return Mono.just(database.get(id));
}
```

Whoever calls this can decide what "empty" should mean to them:

```java
findUser("123")
    .switchIfEmpty(Mono.error(new UserNotFoundException("123")))
    .subscribe(user -> System.out.println(user));
```

## Why It Matters

Keeping "empty" and "error" as two separate ideas matters — an empty result
usually isn't a failure, just a lack of data. Code that treats every "not found"
as an exception ends up harder to read and harder to compose cleanly.
