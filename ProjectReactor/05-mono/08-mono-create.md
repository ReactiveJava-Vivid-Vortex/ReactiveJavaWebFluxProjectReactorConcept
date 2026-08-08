# Mono.create()

## In Simple Terms

`Mono.create(sink -> ...)` gives you full manual control over sending out a
value, an error, or nothing at all, using a `MonoSink`. It's the escape hatch for
plugging **old-style, callback-based APIs** into the reactive world.

## Simple Example

Imagine a legacy API that gives you a callback instead of a `Mono`:

```java
interface LegacyCallback {
    void onSuccess(String result);
    void onFailure(Throwable error);
}

void legacyAsyncCall(LegacyCallback callback) {
    // imagine this calls back asynchronously later
}
```

Wrapping it into a `Mono`:

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

`Mono.create()` is your bridge from "the old world" (callback APIs, legacy SDKs)
into reactive code. Use it sparingly — only when you genuinely have something
non-reactive to connect — since it skips past Reactor's usual automatic demand
handling, and it's on you to correctly call `sink.success()`, `sink.error()`, or
nothing at all, exactly once.
