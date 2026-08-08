# Flux.just()

## In Simple Terms

`Flux.just(...)` creates a `Flux` that sends out a **fixed, already-known set of
values**, then finishes. It's the simplest way to make a small stream out of data
you already have.

## Simple Example

```java
Flux<String> fruits = Flux.just("Apple", "Banana", "Cherry");

fruits.subscribe(
    fruit -> System.out.println("Fruit: " + fruit),
    error -> System.out.println("Error: " + error),
    () -> System.out.println("All fruits emitted!")
);
```

Output:
```
Fruit: Apple
Fruit: Banana
Fruit: Cherry
All fruits emitted!
```

Just like `Mono.just()`, the values are grabbed right away — and `null` values
aren't allowed anywhere in the list.

## Why It Matters

`Flux.just()` is great for quick prototypes, tests, and small fixed lists (like a
hardcoded set of supported currencies). For anything bigger or that comes from a
real collection, use `Flux.fromIterable()` instead.
