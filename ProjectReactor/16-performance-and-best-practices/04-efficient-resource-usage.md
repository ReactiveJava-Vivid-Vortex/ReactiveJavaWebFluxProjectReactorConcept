# Efficient Resource Usage

## In Simple Terms

Beyond just threads, reactive programming — done well — tends to use
memory and connections more wisely overall, since data moves through the
system as it becomes available rather than being fully loaded up front,
and backpressure stops a fast producer from piling up unbounded amounts of
data in memory.

## Simple Example

Inefficient — loading everything into memory before doing anything:

```java
List<Order> allOrders = orderRepository.findAllBlocking(); // loads millions of rows into a List
allOrders.forEach(this::processOrder);
```

Efficient — streaming, roughly constant memory use no matter the dataset
size:

```java
orderRepository.findAll() // returns Flux<Order>, streamed from the database
    .flatMap(this::processOrderReactively, 10) // bounded concurrency
    .subscribe();
```

## Why It Matters

This really matters at scale: a batch job processing a million records
reactively can hold roughly steady memory use the whole time, while a naive
"load it all into a `List` first" approach risks an `OutOfMemoryError` as
data grows. Paired with backpressure, streaming keeps resource usage tied
to how fast you're processing, not how big the dataset happens to be.
