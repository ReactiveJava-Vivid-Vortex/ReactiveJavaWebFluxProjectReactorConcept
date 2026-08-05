# publishOn()

## In Simple Terms

`.publishOn(scheduler)` switches the thread used for **everything downstream** of
this point in the pipeline, starting from the next signal (`onNext`, `onComplete`,
`onError`) that flows through it. It only affects the "downstream" part of the chain
— operators before `.publishOn()` are unaffected.

## Simple Example

```java
Flux.range(1, 3)
    .doOnNext(n -> System.out.println("Before publishOn: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.boundedElastic())
    .doOnNext(n -> System.out.println("After publishOn: " + Thread.currentThread().getName()))
    .subscribe();
```

Output:
```
Before publishOn: main
After publishOn: boundedElastic-1
Before publishOn: main
After publishOn: boundedElastic-1
Before publishOn: main
After publishOn: boundedElastic-1
```

You can use `.publishOn()` multiple times in one chain to switch threads more than
once — e.g., do CPU work on `parallel()`, then switch again to `boundedElastic()` for
a blocking database write.

## Why It Matters

`.publishOn()` gives you fine-grained control over exactly **where in the chain** a
thread switch happens — useful when only part of your pipeline needs to run on a
particular kind of thread (e.g., only the final database write should run on
`boundedElastic()`, while earlier transformations remain on the event loop).
