# doFirst()

## In Simple Terms

`.doFirst()` runs something right at the very start, before the stream even
begins hooking itself up — earlier than any other `doOn*` operator fires.
Most `do*` operators react to something happening as it passes through;
`doFirst()` is different — it just runs the instant `.subscribe()` is called,
before anything else in the chain has had a chance to move.

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

One quirk worth knowing: if you stack up several `.doFirst()` calls in a
chain, they fire in **reverse order** — the last one you wrote runs first.
It rarely matters in everyday code, but it can be confusing if you hit it
unexpectedly.

## Why It Matters

`.doFirst()` is handy for one-time setup that has to happen right before
subscription starts — like kicking off a stopwatch to time the whole
pipeline, or logging "starting operation X" before anything else fires.
