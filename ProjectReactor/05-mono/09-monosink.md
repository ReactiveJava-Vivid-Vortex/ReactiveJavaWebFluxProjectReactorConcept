# MonoSink

## In Simple Terms

`MonoSink<T>` is the object handed to you inside `Mono.create(sink -> ...)` —
your "microphone" for manually announcing exactly one thing: a successful value,
an error, or nothing. Once you use it, the `Mono` is done — anything you call
after that is just ignored.

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

Handling errors too:

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

The rule to remember: call exactly one of `success()`, `success(value)`, or
`error()` — exactly once. Call it twice, and the extras are ignored. Forget to
call it at all (say, because a legacy callback never fires), and the `Mono` just
hangs forever, waiting for something that will never come.
