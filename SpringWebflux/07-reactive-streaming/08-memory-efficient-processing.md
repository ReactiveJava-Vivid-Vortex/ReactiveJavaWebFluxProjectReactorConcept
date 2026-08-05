# Memory-Efficient Processing

## In Simple Terms

"Memory-efficient processing" refers to designing your reactive pipelines so that
memory usage stays roughly **constant**, regardless of how large the underlying
dataset is — achieved by streaming data through the pipeline incrementally instead
of collecting it all into memory-resident structures (`List`, `Map`) at any point.

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

Memory-efficient — processes one order at a time, never holding the full list:

```java
public Mono<BigDecimal> calculateTotalRevenue() {
    return orderRepository.findAll()
        .map(Order::getTotal)
        .reduce(BigDecimal.ZERO, BigDecimal::add); // aggregates as it goes, streaming
}
```

Both produce the same result, but the second version never holds more than one
order's data (plus a running total) in memory at once — critical for datasets too
large to comfortably fit in memory as a `List`.

## Why It Matters

Being deliberate about avoiding unnecessary `.collectList()`/`.collectMap()` calls
(or bounding them with `.buffer()` when some batching is genuinely needed) is what
allows reactive pipelines to process datasets far larger than available memory —
one of the most practically valuable benefits of the streaming model.
