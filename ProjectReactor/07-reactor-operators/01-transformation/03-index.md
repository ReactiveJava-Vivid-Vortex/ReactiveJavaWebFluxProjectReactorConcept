# index()

## In Simple Terms

`.index()` pairs each item in a `Flux` with its **zero-based position** in the
sequence, wrapping each into a `Tuple2<Long, T>` (index, value). It's the reactive
equivalent of Java's `IntStream.range()` combined with an element, or Python's
`enumerate()`.

## Simple Example

```java
Flux.just("Apple", "Banana", "Cherry")
    .index()
    .subscribe(tuple -> System.out.println(tuple.getT1() + ": " + tuple.getT2()));
```

Output:
```
0: Apple
1: Banana
2: Cherry
```

You can also supply a custom index-mapping function:

```java
Flux.just("A", "B", "C")
    .index((idx, value) -> "Item #" + idx + " = " + value)
    .subscribe(System.out::println);
```

Output:
```
Item #0 = A
Item #1 = B
Item #2 = C
```

## Why It Matters

`.index()` is handy whenever you need positional information — e.g., logging
"processing item 5 of N," numbering rows in a generated report, or implementing
simple pagination-like logic within a stream — without needing to maintain a manual
counter variable yourself.
