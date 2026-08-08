# Recovering From Failures

## In Simple Terms

"Recovering from failures" is about designing your pipelines so errors
don't just crash the whole operation — instead, you combine
`onErrorResume`, `retryWhen`, `timeout`, and fallback values to keep things
running, maybe in a scaled-back way, whenever something goes wrong.

## Simple Example

A realistic, layered recovery plan for calling an unreliable external API:

```java
public Mono<ExchangeRate> getExchangeRate(String currency) {
    return exchangeRateApi.getRate(currency)
        .timeout(Duration.ofSeconds(2))
        .retryWhen(Retry.backoff(2, Duration.ofMillis(200)))
        .onErrorResume(error -> {
            log.warn("Exchange rate API failed for {}: {}", currency, error.getMessage());
            return exchangeRateCache.getLastKnownRate(currency);
        })
        .switchIfEmpty(Mono.just(ExchangeRate.defaultFor(currency)));
}
```

This pipeline waits at most 2 seconds, retries a couple of times with a
growing delay if things fail, falls back to a cached rate if the API keeps
failing, and finally falls back to a sensible default if even the cache
comes up empty.

## Why It Matters

Thinking through failure recovery ahead of time is what separates a
fragile system (where one bad dependency drags everything else down with
it) from a resilient one (where failures stay contained and things degrade
gracefully). This is exactly the kind of resilience reactive programming
promises — and Reactor gives you all the pieces to actually build it.
