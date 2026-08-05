# filter()

## In Simple Terms

`.filter(predicate)` only lets through items that satisfy a condition (`predicate`
returns `true`); items that don't match are silently dropped from the stream. It's
the reactive equivalent of `Stream.filter()`.

## Simple Example

```java
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)
    .subscribe(even -> System.out.println("Even: " + even));
```

Output:
```
Even: 2
Even: 4
Even: 6
Even: 8
Even: 10
```

A realistic example: only forwarding orders above a certain value.

```java
orderFlux
    .filter(order -> order.getTotal() > 100)
    .subscribe(order -> System.out.println("High value order: " + order.getId()));
```

## Why It Matters

`.filter()` is one of the most fundamental operators — used constantly to narrow down
a stream to only the items relevant for further processing, without needing manual
`if` checks inside every downstream operator.
