# skip()

## In Simple Terms

`.skip(n)` discards the **first `n` items** from a `Flux` and lets everything after
that through. There's also a `.skip(Duration)` overload that discards items emitted
within a fixed time window from the start.

## Simple Example

```java
Flux.range(1, 10)
    .skip(3)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 4
Got: 5
Got: 6
Got: 7
Got: 8
Got: 9
Got: 10
```

Often combined with `.take()` for simple pagination-like behavior:

```java
// "page 2" of size 5: skip the first 5, take the next 5
Flux.range(1, 20)
    .skip(5)
    .take(5)
    .subscribe(n -> System.out.println("Page item: " + n));
```

## Why It Matters

`.skip()` is a straightforward way to ignore leading items you don't care about —
e.g., skipping a header row in a data feed, or implementing simple in-memory
pagination when combined with `.take()`.
