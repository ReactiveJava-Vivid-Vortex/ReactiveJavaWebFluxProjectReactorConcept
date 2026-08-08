# Factory Methods (Mono)

## In Simple Terms

Reactor gives you a bunch of `Mono.xxx()` methods to create a `Mono` from
different starting points. Picking the right one for the job is the key skill
here.

## Quick Reference

| Factory Method            | Use When...                                             |
|----------------------------|----------------------------------------------------------|
| `Mono.just(value)`          | You already have a value in hand (non-null, runs right away)       |
| `Mono.empty()`              | You want to say "no value, but no error either"             |
| `Mono.error(t)`             | You want to say "this failed"                           |
| `Mono.fromSupplier(fn)`     | Lazy, simple computation, no checked exceptions       |
| `Mono.fromCallable(fn)`     | Lazy, simple computation that might throw a checked exception |
| `Mono.fromRunnable(fn)`     | An action with no return value (`Mono<Void>`) |
| `Mono.defer(fn)`            | You need to choose a whole new Mono, fresh, per subscription  |
| `Mono.create(sink -> ...)`  | Bridging an old-style, callback-based API                |
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

Picking the wrong one causes real bugs — like accidentally running expensive
code right away with `Mono.just(expensiveCall())` instead of lazily with
`Mono.fromSupplier(() -> expensiveCall())`. Knowing this table well saves a lot
of head-scratching later.
