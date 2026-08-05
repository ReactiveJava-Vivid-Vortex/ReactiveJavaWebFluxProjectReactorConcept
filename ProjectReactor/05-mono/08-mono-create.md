# Mono.create()

## In Simple Terms

`Mono.create(sink -> ...)` gives you full manual control over emitting a value, error,
or completion, using a `MonoSink`. It's the escape hatch for bridging **non-reactive,
callback-based APIs** (like an old-style async library that gives you a callback
instead of returning a `Mono`) into the reactive world.

## Simple Example

Imagine a legacy async API that uses callbacks:

```java
interface LegacyCallback {
    void onSuccess(String result);
    void onFailure(Throwable error);
}

void legacyAsyncCall(LegacyCallback callback) {
    // imagine this calls back asynchronously later
}
```

Bridging it into a `Mono`:

```java
Mono<String> mono = Mono.create(sink -> {
    legacyAsyncCall(new LegacyCallback() {
        @Override
        public void onSuccess(String result) {
            sink.success(result);
        }

        @Override
        public void onFailure(Throwable error) {
            sink.error(error);
        }
    });
});

mono.subscribe(result -> System.out.println("Got: " + result));
```

## Why It Matters

`Mono.create()` is your bridge from the "old world" (callback-based APIs, legacy
SDKs) into the reactive world. It should be used **sparingly** — only when you truly
have a non-reactive source to adapt — since it bypasses Reactor's normal automatic
backpressure/demand handling and requires you to correctly call `sink.success()`,
`sink.error()`, or nothing (for empty) exactly once.
