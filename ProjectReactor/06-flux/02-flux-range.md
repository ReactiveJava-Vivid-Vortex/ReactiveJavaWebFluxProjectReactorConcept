# Flux.range()

## In Simple Terms

`Flux.range(start, count)` creates a `Flux<Integer>` that counts out a sequence
of numbers, starting at `start`, for `count` total values. It's a lightweight way
to generate a number sequence without building a `List` first.

## Simple Example

```java
Flux.range(1, 5)
    .subscribe(n -> System.out.println("Number: " + n));
```

Output:
```
Number: 1
Number: 2
Number: 3
Number: 4
Number: 5
```

Handy for generating quick test data:

```java
Flux.range(1, 3)
    .map(n -> "Item-" + n)
    .subscribe(System.out::println);
// Item-1
// Item-2
// Item-3
```

## Why It Matters

`Flux.range()` only produces a number **when asked for it**, respecting
backpressure — it never pre-builds a giant list in memory, even for a huge count.
That makes it a great teaching tool for backpressure, and a handy way to generate
sequences for pagination, retries, or batch indexes.
