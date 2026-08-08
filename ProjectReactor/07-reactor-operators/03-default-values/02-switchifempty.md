# switchIfEmpty()

## In Simple Terms

`.switchIfEmpty()` is like `.defaultIfEmpty()`'s bigger sibling: instead of
handing back one fixed value when the stream comes up empty, it switches
over to an entirely different `Mono`/`Flux` — which can itself go do more
work, like calling a database. Think of it as "plan B," where plan B is a
whole separate action, not just a spare value sitting in your pocket.

## Simple Example

```java
public Mono<User> findUser(String id) {
    return primaryDatabase.findById(id)
        .switchIfEmpty(secondaryDatabase.findById(id)); // fallback to a second lookup
}
```

If the primary lookup comes back empty, the secondary lookup kicks in — and
whatever it returns (or its own empty/error outcome) becomes the final
result.

```java
Mono.<String>empty()
    .switchIfEmpty(Mono.defer(() -> {
        System.out.println("Primary empty, trying fallback...");
        return Mono.just("Fallback value");
    }))
    .subscribe(System.out::println);

// Output:
// Primary empty, trying fallback...
// Fallback value
```

## Why It Matters

`.switchIfEmpty()` shows up all the time in real systems: check a fast cache
first, and only fall back to a slower database if the cache comes up empty
— all written as one clean chain, with no nested if/else and no blocking
calls.

```java
cache.get(key)
    .switchIfEmpty(database.findByKey(key).doOnNext(value -> cache.put(key, value)))
    .subscribe(value -> System.out.println("Value: " + value));
```
