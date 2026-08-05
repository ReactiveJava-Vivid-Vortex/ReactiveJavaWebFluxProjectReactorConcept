# Logging

## In Simple Terms

Debugging reactive pipelines can be tricky because nothing happens until someone
subscribes, and the code doesn't execute top-to-bottom like normal imperative code.
Project Reactor provides a built-in `.log()` operator that prints every signal
(`onSubscribe`, `request`, `onNext`, `onComplete`, `onError`) flowing through that
point in the pipeline — extremely useful for understanding what's actually happening.

## Simple Example

```java
Flux.just(1, 2, 3)
    .log()
    .map(n -> n * 2)
    .subscribe(System.out::println);
```

Output (abbreviated):
```
[ INFO] onSubscribe(FluxArray.ArraySubscription)
[ INFO] request(unbounded)
[ INFO] onNext(1)
[ INFO] onNext(2)
[ INFO] onNext(3)
[ INFO] onComplete()
2
4
6
```

You can also give the log a name to distinguish multiple `.log()` calls in a longer
pipeline:

```java
Flux.just(1, 2, 3)
    .log("before-map")
    .map(n -> n * 2)
    .log("after-map")
    .subscribe();
```

## Why It Matters

`.log()` is often the fastest way to understand *why* your pipeline isn't behaving as
expected — did it even get subscribed to? Did the request demand ever reach the
source? Did an error happen before or after a particular operator? It's the reactive
equivalent of sprinkling `System.out.println()` through imperative code, but built
specifically to show Reactive Streams signals.
