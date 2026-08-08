# Producer APIs

## In Simple Terms

`Sinks` give you a small, focused set of methods for actually pushing data
into the reactive world by hand. Knowing exactly what these methods promise
you — especially their `EmitResult` return values — is the key to using
sinks correctly and safely.

## Key Methods

```java
Sinks.Many<Integer> sink = Sinks.many().multicast().onBackpressureBuffer();

Sinks.EmitResult result = sink.tryEmitNext(42);   // attempt to emit a value
sink.tryEmitComplete();                            // signal successful completion
sink.tryEmitError(new RuntimeException("failed")); // signal failure
```

Each `tryEmitXxx()` call hands you back an `EmitResult` instead of throwing
an exception on failure — a value you can check and decide what to do
about:

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

For simpler cases where you know for sure only one thread is calling in,
there's also a convenience method:

```java
sink.emitNext(42, Sinks.EmitFailureHandler.FAIL_FAST); // throws on failure instead
```

## Why It Matters

This "hand back a result instead of throwing" contract is exactly what
makes `Sinks` safer than the old `Processor` API, which could blow up with
unpredictable exceptions if misused. Always check (or deliberately handle)
the `EmitResult` you get back — quietly ignoring a `FAIL_OVERFLOW` or
`FAIL_TERMINATED` can hide real bugs, like events silently going missing.
