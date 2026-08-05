# MonoSink

## In Simple Terms

`MonoSink<T>` is the object handed to you inside `Mono.create(sink -> ...)` — it's
your "microphone" for manually emitting exactly one signal: a success value, an
error, or nothing (empty). Once you call one of its methods, the `Mono` is done —
further calls are ignored.

```java
public interface MonoSink<T> {
    void success();
    void success(T value);
    void error(Throwable e);
    // ... plus a few advanced hooks like onDispose(), onCancel()
}
```

## Simple Example

```java
Mono<String> mono = Mono.create(sink -> {
    boolean found = performLookup();

    if (found) {
        sink.success("Found it!");
    } else {
        sink.success(); // completes empty, like Mono.empty()
    }
});
```

Handling potential errors as well:

```java
Mono<Integer> divideMono = Mono.create(sink -> {
    try {
        int result = 10 / getDivisor();
        sink.success(result);
    } catch (ArithmeticException e) {
        sink.error(e);
    }
});
```

## Why It Matters

Understanding `MonoSink`'s contract (call exactly one of `success()`, `success(T)`, or
`error()`, exactly once) is essential when bridging non-reactive callback APIs into
`Mono`. Calling it multiple times, or forgetting to call it entirely (e.g., a
callback that's never invoked by the legacy library), leads to a `Mono` that either
misbehaves or hangs forever.
