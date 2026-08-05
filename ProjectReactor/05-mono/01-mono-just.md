# Mono.just()

## In Simple Terms

`Mono.just(value)` creates a `Mono` that emits **exactly one, already-known value**
immediately when subscribed to. It's the simplest way to wrap a value you already
have into a reactive type.

**Important gotcha:** the value must not be `null` — `Mono.just(null)` throws a
`NullPointerException` immediately, because Reactive Streams forbids `null` as a
valid element. Use `Mono.justOrEmpty()` if the value might be `null`.

## Simple Example

```java
Mono<String> mono = Mono.just("Hello, Reactor!");

mono.subscribe(value -> System.out.println("Got: " + value));
// Output: Got: Hello, Reactor!
```

Since the value is eagerly captured at creation time (not lazily computed), be
careful with expensive calls:

```java
// BAD: fetchFromDatabase() runs immediately when this line executes,
// even before anyone subscribes!
Mono<User> mono = Mono.just(fetchFromDatabase());
```

For lazy, deferred computation, use `Mono.fromSupplier()` or `Mono.defer()` instead.

## Why It Matters

`Mono.just()` is extremely common for wrapping constants, test fixtures, or values
already available in memory (e.g., a default fallback value) into the reactive world
so they compose cleanly with other `Mono`/`Flux` pipelines.
