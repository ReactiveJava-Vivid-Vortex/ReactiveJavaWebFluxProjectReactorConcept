# Producer APIs

## In Simple Terms

`Sinks` expose a small, focused **producer API** — the methods you call to actually
push data into the reactive world manually. Understanding the exact contract of
these methods (especially their `EmitResult` return values) is key to using sinks
correctly and safely.

## Key Methods

```java
Sinks.Many<Integer> sink = Sinks.many().multicast().onBackpressureBuffer();

Sinks.EmitResult result = sink.tryEmitNext(42);   // attempt to emit a value
sink.tryEmitComplete();                            // signal successful completion
sink.tryEmitError(new RuntimeException("failed")); // signal failure
```

Each `tryEmitXxx()` call returns an `EmitResult` — instead of throwing an exception on
failure, it returns a value you can inspect and react to:

```java
Sinks.EmitResult result = sink.tryEmitNext(42);

if (result.isFailure()) {
    switch (result) {
        case FAIL_OVERFLOW -> System.out.println("Buffer full, item dropped");
        case FAIL_TERMINATED -> System.out.println("Sink already completed");
        case FAIL_NON_SERIALIZED -> System.out.println("Concurrent emission without proper serialization");
        default -> System.out.println("Emission failed: " + result);
    }
}
```

For simpler cases where you're certain about single-threaded access, there's also a
convenience method:

```java
sink.emitNext(42, Sinks.EmitFailureHandler.FAIL_FAST); // throws on failure instead
```

## Why It Matters

Understanding the producer API's explicit, non-throwing `EmitResult` contract is what
makes `Sinks` safer than the older `Processor` API (which could throw unpredictable
exceptions on misuse). Always check (or explicitly handle) the `EmitResult` rather
than ignoring it — silently ignoring a `FAIL_OVERFLOW` or `FAIL_TERMINATED` result can
hide real bugs, like silently dropped events.
