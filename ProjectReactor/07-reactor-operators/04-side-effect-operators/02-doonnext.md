# doOnNext()

## In Simple Terms

`.doOnNext()` lets you peek at each item as it flows by and do something
with it — like log it — without touching or changing the item itself. It's
a security camera, not a checkpoint: it watches, it doesn't interfere. The
value keeps moving on to the next step exactly as it was.

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

Very commonly used to log or track things at different stages of a pipeline:

```java
orderFlux
    .doOnNext(order -> log.info("Received order: {}", order.getId()))
    .filter(order -> order.getTotal() > 0)
    .doOnNext(order -> log.info("Order passed validation: {}", order.getId()))
    .subscribe(orderService::process);
```

## Why It Matters

`.doOnNext()` is your standard way to add visibility — logging, metrics,
tracing — into a pipeline without changing what actually flows through it.
It's a window you can open at any point in the chain to see what's going by.
