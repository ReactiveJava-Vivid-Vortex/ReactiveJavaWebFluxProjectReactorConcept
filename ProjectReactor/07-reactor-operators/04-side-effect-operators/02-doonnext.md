# doOnNext()

## In Simple Terms

`.doOnNext(consumer)` lets you run a side effect (like logging) every time an item
passes through this point in the pipeline, **without modifying the item itself**.
The value is passed through unchanged to the next operator.

## Simple Example

```java
Flux.just(1, 2, 3)
    .doOnNext(n -> System.out.println("About to process: " + n))
    .map(n -> n * 10)
    .subscribe(result -> System.out.println("Result: " + result));
```

Output:
```
About to process: 1
Result: 10
About to process: 2
Result: 20
About to process: 3
Result: 30
```

Very commonly used for logging or metrics at multiple stages of a pipeline:

```java
orderFlux
    .doOnNext(order -> log.info("Received order: {}", order.getId()))
    .filter(order -> order.getTotal() > 0)
    .doOnNext(order -> log.info("Order passed validation: {}", order.getId()))
    .subscribe(orderService::process);
```

## Why It Matters

`.doOnNext()` is the standard way to add observability (logging, metrics, tracing)
into a reactive pipeline without altering its actual data flow — a "read-only tap"
into the stream at any point you choose.
