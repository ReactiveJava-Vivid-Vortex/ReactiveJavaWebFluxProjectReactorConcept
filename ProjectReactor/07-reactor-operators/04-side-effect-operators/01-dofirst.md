# doFirst()

## In Simple Terms

`.doFirst(runnable)` runs a side-effecting action **before** the subscription process
even begins — it fires earlier than `doOnSubscribe()`. Unlike most `doOn*` operators
which react to a signal as it passes through, `doFirst()` runs synchronously right at
the moment `.subscribe()` is called, before anything else in the chain executes.

## Simple Example

```java
Mono.just("Hello")
    .doFirst(() -> System.out.println("1. About to subscribe"))
    .doOnSubscribe(s -> System.out.println("2. Subscribed"))
    .doOnNext(v -> System.out.println("3. Value: " + v))
    .subscribe();
```

Output:
```
1. About to subscribe
2. Subscribed
3. Value: Hello
```

Interestingly, if you have multiple `.doFirst()` calls in a chain, they execute in
**reverse** order relative to their position (the last one added in the chain runs
first) — this is a quirk worth knowing about, though it rarely matters in typical use.

## Why It Matters

`.doFirst()` is useful for setup logic that must run exactly once, right before
subscription kicks off — like starting a timer to measure total pipeline duration, or
logging "starting operation X" before any other signal fires.
