# Mono.just()

## In Simple Terms

`Mono.just(value)` wraps a value you **already have** into a `Mono` that hands it
out the instant someone subscribes. It's the simplest possible way to turn a
plain value into a reactive one.

**Watch out:** the value can't be `null` — `Mono.just(null)` blows up with a
`NullPointerException` right away, because Reactive Streams doesn't allow `null`
as a value. If the value might be `null`, use `Mono.justOrEmpty()` instead.

## Simple Example

```java
Mono<String> mono = Mono.just("Hello, Reactor!");

mono.subscribe(value -> System.out.println("Got: " + value));
// Output: Got: Hello, Reactor!
```

The value gets grabbed right away, not lazily — so be careful with anything
expensive:

```java
// BAD: fetchFromDatabase() runs THE MOMENT this line executes,
// even if nobody ever subscribes!
Mono<User> mono = Mono.just(fetchFromDatabase());
```

If you need the value computed lazily, use `Mono.fromSupplier()` or
`Mono.defer()` instead.

## Why It Matters

`Mono.just()` is great for wrapping constants, test values, or anything already
sitting in memory (like a default fallback) so it fits neatly into the rest of
your `Mono`/`Flux` pipeline.
