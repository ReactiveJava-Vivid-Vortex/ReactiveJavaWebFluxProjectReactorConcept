# log()

## In Simple Terms

`.log()` is a diagnostic operator that prints (via SLF4J, at INFO level by default)
every Reactive Streams signal passing through this point in the pipeline —
`onSubscribe`, `request`, `onNext`, `onComplete`, `onError`. It's the most useful
tool for understanding what's actually happening inside a reactive pipeline.

## Simple Example

```java
Flux.range(1, 3)
    .log()
    .map(n -> n * 2)
    .subscribe();
```

Output (abbreviated):
```
[ INFO] onSubscribe(FluxRange.RangeSubscription)
[ INFO] request(unbounded)
[ INFO] onNext(1)
[ INFO] onNext(2)
[ INFO] onNext(3)
[ INFO] onComplete()
```

You can name multiple `.log()` calls in the same pipeline to tell them apart:

```java
Flux.range(1, 3)
    .log("source")
    .filter(n -> n % 2 == 0)
    .log("after-filter")
    .subscribe();
```

## Why It Matters

`.log()` is often the fastest way to diagnose a reactive pipeline that isn't behaving
as expected: Did it get subscribed to? How much was requested? Did an error occur
before or after a specific operator? It's the reactive-world equivalent of
strategically placed `System.out.println()` calls, but far more informative.
