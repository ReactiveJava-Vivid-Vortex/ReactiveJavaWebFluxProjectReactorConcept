# Logging (Mono)

## In Simple Terms

Just like with `Flux`, you can stick `.log()` onto a `Mono` pipeline to see
exactly what's happening — great for figuring out subscription timing, whether a
value actually came through, and if/when an error happened.

## Simple Example

```java
Mono.just("Hello")
    .log()
    .map(String::toUpperCase)
    .subscribe();
```

Output (shortened):
```
[ INFO] onSubscribe([Fuseable] Operators.ScalarSubscription)
[ INFO] request(unbounded)
[ INFO] onNext(Hello)
[ INFO] onComplete()
```

Compare that to an empty `Mono`'s log output:

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

When you're trying to figure out why a `Mono` doesn't seem to be giving you
anything, `.log()` tells you right away whether it finished empty, errored out,
or was never even subscribed to — much faster than guessing with print
statements.
