# Logging

## In Simple Terms

Reactive code can be confusing to debug, because nothing runs until someone
subscribes, and it doesn't execute top-to-bottom the way normal code does.
Project Reactor's `.log()` operator fixes this by printing out every single
signal (`onSubscribe`, `request`, `onNext`, `onComplete`, `onError`) as it passes
through that point in your pipeline — so you can actually see what's happening.

## Simple Example

```java
Flux.just(1, 2, 3)
    .log()
    .map(n -> n * 2)
    .subscribe(System.out::println);
```

Output (shortened):
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

You can also name each `.log()` call if you have several in one pipeline, so you
can tell them apart:

```java
Flux.just(1, 2, 3)
    .log("before-map")
    .map(n -> n * 2)
    .log("after-map")
    .subscribe();
```

## Why It Matters

`.log()` is usually the quickest way to figure out why a pipeline isn't doing
what you expect. Did it even get subscribed to? Did the request for data actually
reach the source? Did an error happen before or after a certain step? It's the
reactive version of scattering `System.out.println()` everywhere — just built
specifically for reactive signals.
