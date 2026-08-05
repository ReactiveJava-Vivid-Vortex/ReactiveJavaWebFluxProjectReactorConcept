# Logging (Mono)

## In Simple Terms

Just like with `Flux`, you can attach `.log()` to a `Mono` pipeline to see exactly
what signals fire and when — extremely useful for understanding subscription timing,
whether a value was emitted, or if/when an error occurred.

## Simple Example

```java
Mono.just("Hello")
    .log()
    .map(String::toUpperCase)
    .subscribe();
```

Output (abbreviated):
```
[ INFO] onSubscribe([Fuseable] Operators.ScalarSubscription)
[ INFO] request(unbounded)
[ INFO] onNext(Hello)
[ INFO] onComplete()
```

You can compare an empty `Mono`'s log output to see the difference:

```java
Mono.empty().log().subscribe();
```
Output:
```
[ INFO] onSubscribe(...)
[ INFO] request(unbounded)
[ INFO] onComplete()   <- no onNext() line at all!
```

## Why It Matters

When debugging why a `Mono` seems to "not emit anything," `.log()` immediately tells
you whether it's because the `Mono` completed empty, errored out, or was never even
subscribed to in the first place — saving a lot of guesswork compared to sprinkling
manual print statements.
