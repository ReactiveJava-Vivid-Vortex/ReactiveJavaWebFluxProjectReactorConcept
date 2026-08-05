# Efficient Resource Usage

## In Simple Terms

Beyond just threads, reactive programming (when done correctly) tends to use memory
and connections more efficiently overall — because data flows through the system as
it becomes available, rather than being fully buffered/loaded upfront, and because
backpressure prevents unbounded memory growth from fast producers.

## Simple Example

Inefficient — loading everything into memory before processing:

```java
List<Order> allOrders = orderRepository.findAllBlocking(); // loads millions of rows into a List
allOrders.forEach(this::processOrder);
```

Efficient — streaming, constant memory usage regardless of dataset size:

```java
orderRepository.findAll() // returns Flux<Order>, streamed from the database
    .flatMap(this::processOrderReactively, 10) // bounded concurrency
    .subscribe();
```

## Why It Matters

Efficient resource usage matters most at scale: a batch job processing a million
records reactively can maintain roughly constant memory usage throughout, whereas a
naive "load everything into a `List` first" approach risks `OutOfMemoryError` as
data volumes grow. Combined with backpressure, streaming keeps resource usage
proportional to processing speed, not dataset size.
