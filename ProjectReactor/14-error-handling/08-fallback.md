# Fallback

## In Simple Terms

"Fallback" is the general pattern of providing an alternate value or behavior when
the primary operation fails — the umbrella concept behind `.onErrorReturn()`,
`.onErrorResume()`, and `.defaultIfEmpty()`. Good reactive error handling almost
always involves designing what the *fallback* should be for each failure scenario.

## Simple Example

Combining multiple fallback layers for a robust pipeline:

```java
public Mono<Price> getPrice(String productId) {
    return primaryPriceService.getPrice(productId)
        .timeout(Duration.ofSeconds(1))
        .onErrorResume(error -> cachedPriceService.getPrice(productId)) // fallback #1: cache
        .switchIfEmpty(Mono.just(Price.DEFAULT))                        // fallback #2: default
        .onErrorReturn(Price.UNAVAILABLE);                               // fallback #3: last resort
}
```

This layers fallbacks: try the live service (with a timeout), fall back to a cache if
that fails, fall back to a sensible default if there's genuinely no data, and as an
absolute last resort, return a static "unavailable" marker rather than propagating
any error to the caller.

## Why It Matters

Designing deliberate fallback strategies — rather than letting errors simply
propagate — is what makes reactive microservices **resilient**. A well-designed
fallback chain means a single failing dependency degrades gracefully instead of
cascading into a full outage for the caller.
