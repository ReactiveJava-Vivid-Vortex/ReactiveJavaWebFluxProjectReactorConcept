# subscribeOn()

## In Simple Terms

`.subscribeOn()` controls which thread the whole thing *starts* on — no
matter where you put it in the chain, it reaches all the way back to the
very beginning and affects the entire pipeline. That's the key difference
from `.publishOn()`, which only affects what comes after it.

## Simple Example

```java
Mono.fromCallable(() -> {
    System.out.println("Source running on: " + Thread.currentThread().getName());
    return "data";
})
.subscribeOn(Schedulers.boundedElastic())
.doOnNext(v -> System.out.println("map running on: " + Thread.currentThread().getName()))
.subscribe();
```

Output:
```
Source running on: boundedElastic-1
map running on: boundedElastic-1
```

**Key distinction:**

| Operator        | Affects...                                         |
|-------------------|-----------------------------------------------------|
| `.subscribeOn()`   | The whole chain, from the very source, regardless of where it's placed |
| `.publishOn()`     | Only what comes AFTER it in the chain               |

Even putting `.subscribeOn()` at the very end still reaches back and
affects the source at the start:

```java
Mono.fromCallable(() -> "data")
    .doOnNext(v -> System.out.println("Runs on: " + Thread.currentThread().getName()))
    .subscribeOn(Schedulers.boundedElastic()) // still affects the WHOLE chain
    .subscribe();
```

## Why It Matters

`.subscribeOn()` is the tool for when the *source itself* is blocking (a
blocking database call, reading a file) and needs to start life on the
right kind of thread from the very first moment. Only one `.subscribeOn()`
per chain actually does anything — the first one Reactor sees wins — unlike
`.publishOn()`, which you can use as many times as you like.
