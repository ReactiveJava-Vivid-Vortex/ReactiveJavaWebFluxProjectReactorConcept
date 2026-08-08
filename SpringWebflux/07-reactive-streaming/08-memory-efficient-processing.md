# Memory-Efficient Processing

## In Simple Terms

"Memory-efficient processing" is about designing your pipelines so memory
use stays roughly flat no matter how big the underlying data is —
achieved by streaming data through the pipeline bit by bit instead of
gathering it all into memory-heavy structures (`List`, `Map`) at any
point.

## Simple Example

Memory-inefficient — loads everything into memory:

```java
public Mono<BigDecimal> calculateTotalRevenue() {
    return orderRepository.findAll()
        .collectList()                          // loads ALL orders into memory
        .map(orders -> orders.stream()
            .map(Order::getTotal)
            .reduce(BigDecimal.ZERO, BigDecimal::add));
}
```

Memory-efficient — handles one order at a time, never holding the full
list:

```java
public Mono<BigDecimal> calculateTotalRevenue() {
    return orderRepository.findAll()
        .map(Order::getTotal)
        .reduce(BigDecimal.ZERO, BigDecimal::add); // aggregates as it goes, streaming
}
```

Both give you the same answer, but the second version never holds more
than one order's data (plus a running total) in memory at a time —
critical for datasets too big to comfortably fit in memory as a `List`.

## Why It Matters

Being deliberate about avoiding unnecessary `.collectList()`/`.collectMap()`
calls (or putting explicit bounds on them with `.buffer()` when some
batching is genuinely needed) is what lets reactive pipelines handle
datasets far bigger than available memory — one of the most practically
useful benefits of the streaming approach.
