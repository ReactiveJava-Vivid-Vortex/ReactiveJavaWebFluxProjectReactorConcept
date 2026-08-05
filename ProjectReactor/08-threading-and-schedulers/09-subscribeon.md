# subscribeOn()

## In Simple Terms

`.subscribeOn(scheduler)` controls which thread the **subscription itself** (and the
original source's emission) happens on — regardless of where `.subscribeOn()` is
placed in the chain, it affects the **entire pipeline from the very beginning**. This
is different from `.publishOn()`, which only affects everything after its position.

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

Even placing `.subscribeOn()` at the end of a chain affects the source at the start:

```java
Mono.fromCallable(() -> "data")
    .doOnNext(v -> System.out.println("Runs on: " + Thread.currentThread().getName()))
    .subscribeOn(Schedulers.boundedElastic()) // still affects the WHOLE chain
    .subscribe();
```

## Why It Matters

`.subscribeOn()` is the right tool when the **source itself** is blocking (e.g., a
blocking JDBC call, file read) and needs to run on an appropriate scheduler from the
very start — only one `.subscribeOn()` per chain has any effect (the first one
encountered wins), unlike `.publishOn()` which can be used multiple times.
