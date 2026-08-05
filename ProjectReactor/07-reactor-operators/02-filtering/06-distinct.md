# distinct()

## In Simple Terms

`.distinct()` filters out duplicate items from a `Flux`, letting through only the
first occurrence of each unique value (based on `equals()`/`hashCode()`, or a custom
key extractor you provide). It's the reactive equivalent of removing duplicates from
a list.

## Simple Example

```java
Flux.just(1, 2, 2, 3, 1, 4, 3)
    .distinct()
    .subscribe(n -> System.out.println("Unique: " + n));
```

Output:
```
Unique: 1
Unique: 2
Unique: 3
Unique: 4
```

Using a custom key selector — e.g., deduplicating orders by customer ID instead of
the whole object:

```java
Flux.just(order1, order2, order3)
    .distinct(Order::getCustomerId) // dedupe based on this key
    .subscribe(order -> System.out.println("First order per customer: " + order));
```

**Note:** `.distinct()` keeps track of every unique value it has seen so far in
memory (to detect future duplicates), so it's not suitable for extremely large or
infinite streams with high cardinality — memory usage grows with the number of
distinct values seen.

## Why It Matters

`.distinct()` is a handy, concise way to deduplicate data flowing through a pipeline
— common when merging multiple sources that might contain overlapping records (e.g.,
combining results from a cache and a database).
