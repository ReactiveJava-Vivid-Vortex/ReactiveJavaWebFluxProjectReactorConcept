# Factory Methods (Mono)

## In Simple Terms

Project Reactor gives you many `Mono.xxx()` static factory methods to create a `Mono`
from different kinds of sources. Knowing which one fits your situation is key to
writing correct, efficient reactive code.

## Quick Reference

| Factory Method            | Use When...                                             |
|----------------------------|----------------------------------------------------------|
| `Mono.just(value)`          | You already have a value in hand (non-null, eager)       |
| `Mono.empty()`              | You want to represent "no value" successfully             |
| `Mono.error(t)`             | You want to represent a failure                           |
| `Mono.fromSupplier(fn)`     | Lazy, synchronous computation, no checked exceptions       |
| `Mono.fromCallable(fn)`     | Lazy, synchronous computation that may throw checked exceptions |
| `Mono.fromRunnable(fn)`     | A side-effecting action with no return value (`Mono<Void>`) |
| `Mono.defer(fn)`            | You need to choose/build a whole new Mono per subscription  |
| `Mono.create(sink -> ...)`  | Bridging a callback-based, non-reactive API                |
| `Mono.fromFuture(future)`   | Wrapping a `CompletableFuture`                              |
| `Mono.justOrEmpty(value)`   | A value that might legitimately be `null`                  |

## Simple Example

```java
Mono<String> a = Mono.just("value");
Mono<String> b = Mono.empty();
Mono<String> c = Mono.error(new RuntimeException("oops"));
Mono<String> d = Mono.fromSupplier(() -> computeSomething());
Mono<String> e = Mono.fromCallable(() -> Files.readString(Path.of("file.txt")));
Mono<Void>   f = Mono.fromRunnable(() -> System.out.println("side effect"));
Mono<String> g = Mono.defer(() -> chooseMonoAtRuntime());
Mono<String> h = Mono.justOrEmpty(possiblyNullValue);
```

## Why It Matters

Picking the *right* factory method avoids subtle bugs — like accidentally running
expensive code eagerly with `Mono.just(expensiveCall())` instead of lazily with
`Mono.fromSupplier(() -> expensiveCall())`. Knowing this table by heart saves a lot of
debugging time later.
