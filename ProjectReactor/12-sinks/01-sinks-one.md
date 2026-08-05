# Sinks.One

## In Simple Terms

`Sinks.One<T>` is the modern, safe way to manually produce a **single value** (like a
programmatic `Mono`), replacing older, more error-prone approaches like
`MonoProcessor`. You create it, hold onto it, and call `tryEmitValue()` /
`tryEmitError()` from anywhere in your code whenever the result becomes available.

## Simple Example

```java
Sinks.One<String> sink = Sinks.one();

Mono<String> mono = sink.asMono();
mono.subscribe(value -> System.out.println("Received: " + value));

// Later, from anywhere in your code:
sink.tryEmitValue("Hello from the sink!");
```

Output:
```
Received: Hello from the sink!
```

Emitting an error instead:

```java
Sinks.One<String> errorSink = Sinks.one();
errorSink.asMono().subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e.getMessage())
);
errorSink.tryEmitError(new RuntimeException("Something failed"));
```

**Important:** a `Sinks.One` can only be completed **once** — calling
`tryEmitValue()` a second time returns a failure result (`EmitResult.FAIL_TERMINATED`)
instead of throwing, giving you a chance to handle it explicitly.

## Why It Matters

`Sinks.One` is the correct, modern tool for bridging an external, single-result
callback API into a `Mono` — safer and clearer than the deprecated `MonoProcessor`,
with explicit `EmitResult` return values instead of silent failures.
