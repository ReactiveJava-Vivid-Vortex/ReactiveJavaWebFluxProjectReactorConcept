# Sinks.One

## In Simple Terms

`Sinks.One<T>` is the modern, safe way to manually produce a single value —
think of it as a `Mono` you control by hand. You create it, hang onto it,
and whenever the result is ready — from anywhere in your code — you call
`tryEmitValue()` or `tryEmitError()` to deliver it. It replaces older,
trickier tools like `MonoProcessor`.

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

**Good to know:** a `Sinks.One` can only be completed once — calling
`tryEmitValue()` a second time doesn't throw, it just quietly returns a
failure result (`EmitResult.FAIL_TERMINATED`) that you can check for.

## Why It Matters

`Sinks.One` is the right tool for bridging some external, callback-based
API into a `Mono` — safer and clearer than the old `MonoProcessor`, since it
gives you an explicit answer back instead of silently failing.
