# handle()

## In Simple Terms

`.handle((value, sink) -> ...)` is a flexible operator that combines the abilities of
`map()` and `filter()` into one: for each item, you can choose to emit a transformed
value (`sink.next(...)`), emit nothing (skip the item, like `filter`), or signal an
error (`sink.error(...)`) — all in one place.

## Simple Example

```java
Flux.just(1, 2, 3, 4, 5, 6)
    .handle((n, sink) -> {
        if (n % 2 == 0) {
            sink.next(n * 10); // transform and emit
        }
        // odd numbers are simply skipped (no sink.next() call)
    })
    .subscribe(value -> System.out.println("Got: " + value));
```

Output:
```
Got: 20
Got: 40
Got: 60
```

This is equivalent to `.filter(n -> n % 2 == 0).map(n -> n * 10)`, but done in a
single pass with one operator.

## Why It Matters

`.handle()` is useful when filter-and-map logic is tightly coupled (e.g., you need to
inspect the value to decide both whether to keep it *and* how to transform it), or
when you want to signal a custom error partway through processing based on a
specific item's value — all without chaining multiple separate operators.
