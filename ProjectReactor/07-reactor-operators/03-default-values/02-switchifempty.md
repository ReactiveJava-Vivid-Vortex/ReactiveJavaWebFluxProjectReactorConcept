# switchIfEmpty()

## In Simple Terms

`.switchIfEmpty(alternativePublisher)` is like `.defaultIfEmpty()`, but instead of a
single static fallback value, it switches to an **entirely different Mono/Flux** (which
could itself be asynchronous — e.g., a database fallback query) when the upstream
completes empty.

## Simple Example

```java
public Mono<User> findUser(String id) {
    return primaryDatabase.findById(id)
        .switchIfEmpty(secondaryDatabase.findById(id)); // fallback to a second lookup
}
```

If the primary lookup finds nothing, the secondary lookup is triggered — and its
result (or its own empty/error outcome) becomes the final result.

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

`.switchIfEmpty()` is a very common real-world pattern: try a fast cache lookup first,
and only fall back to a slower database call if the cache misses — all cleanly
expressed in a single reactive chain, without nested if/else or blocking calls.

```java
cache.get(key)
    .switchIfEmpty(database.findByKey(key).doOnNext(value -> cache.put(key, value)))
    .subscribe(value -> System.out.println("Value: " + value));
```
