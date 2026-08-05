# Flux.just()

## In Simple Terms

`Flux.just(...)` creates a `Flux` that emits a **fixed, known set of values** (up to
10 overloaded arguments, or use the varargs form for more), then completes. It's the
simplest way to create a small, finite stream of already-known data.

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

Like `Mono.just()`, the values are captured eagerly — no `null` values are allowed
inside the argument list.

## Why It Matters

`Flux.just()` is perfect for quick prototyping, unit tests, and small fixed
collections (e.g., a hardcoded list of supported currencies). For larger or
dynamically-sized collections, prefer `Flux.fromIterable()` instead.
