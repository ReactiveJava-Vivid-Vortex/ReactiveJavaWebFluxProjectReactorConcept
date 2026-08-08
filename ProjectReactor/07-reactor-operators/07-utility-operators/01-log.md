# log()

## In Simple Terms

`.log()` prints out everything happening at this point in the pipeline —
when someone subscribed, how many items were asked for, every item that
went by, and how the stream ended. It's like putting a security camera with
a running commentary at one spot in your pipeline, so you can actually see
what's going on instead of guessing.

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

You can label multiple `.log()` calls in the same pipeline so you can tell
which one is which:

```java
Flux.range(1, 3)
    .log("source")
    .filter(n -> n % 2 == 0)
    .log("after-filter")
    .subscribe();
```

## Why It Matters

`.log()` is often the quickest way to figure out why a reactive pipeline
isn't doing what you expect: Did anything even subscribe? How much was
requested? Did the error happen before or after a certain step? It's the
reactive world's version of scattering `System.out.println()` everywhere —
except far more informative and much less messy.
