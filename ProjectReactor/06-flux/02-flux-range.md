# Flux.range()

## In Simple Terms

`Flux.range(start, count)` creates a `Flux<Integer>` that emits a sequence of
consecutive integers, starting at `start`, emitting `count` total values. It's a
lazy, memory-efficient way to generate a numeric sequence without pre-building a
`List`.

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

Combined with other operators, it's great for generating test data or simple loops:

```java
Flux.range(1, 3)
    .map(n -> "Item-" + n)
    .subscribe(System.out::println);
// Item-1
// Item-2
// Item-3
```

## Why It Matters

`Flux.range()` produces values **on demand**, respecting backpressure — it doesn't
pre-compute a giant list in memory even if `count` is very large. This makes it
excellent for demonstrating backpressure/demand concepts and for generating simple
sequences for pagination, retries, or batch indices.
