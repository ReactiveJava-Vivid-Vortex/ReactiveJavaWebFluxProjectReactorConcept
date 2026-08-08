# Fallback

## In Simple Terms

"Fallback" is just the general idea of having a plan B ready when the main
thing fails — the umbrella concept behind `.onErrorReturn()`,
`.onErrorResume()`, and `.defaultIfEmpty()`. Solid error handling almost
always comes down to deciding, ahead of time, what the fallback should be
for each way things could go wrong.

## Simple Example

Layering several fallbacks together for a sturdy pipeline:

```java
public Mono<Price> getPrice(String productId) {
    return primaryPriceService.getPrice(productId)
        .timeout(Duration.ofSeconds(1))
        .onErrorResume(error -> cachedPriceService.getPrice(productId)) // fallback #1: cache
        .switchIfEmpty(Mono.just(Price.DEFAULT))                        // fallback #2: default
        .onErrorReturn(Price.UNAVAILABLE);                               // fallback #3: last resort
}
```

This stacks up fallbacks: try the live service (with a timeout), fall back
to a cache if that fails, fall back to a sensible default if there's
genuinely no data, and as a last resort, hand back an "unavailable" marker
instead of ever throwing an error at the caller.

## Why It Matters

Deliberately planning fallbacks — instead of just letting errors bubble up
— is what makes reactive services genuinely resilient. A good fallback
chain means one failing dependency degrades gracefully instead of turning
into a full outage for whoever's calling you.
