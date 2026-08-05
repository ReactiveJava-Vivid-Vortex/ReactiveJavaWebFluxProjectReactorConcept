# Recovering From Failures

## In Simple Terms

"Recovering from failures" means designing your reactive pipelines so that errors
don't simply propagate and crash the whole operation — instead, you use the right
combination of `onErrorResume`, `retryWhen`, `timeout`, and fallback values to keep
the system functioning, in a degraded but acceptable way, when something goes wrong.

## Simple Example

A realistic, layered recovery strategy for calling an unreliable external API:

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

This pipeline: waits at most 2 seconds, retries transient failures with backoff twice,
falls back to a cached rate if the API keeps failing, and finally falls back to a
sensible default if even the cache has nothing.

## Why It Matters

Thoughtful failure recovery is what separates a fragile system (one bad dependency
takes down everything) from a resilient one (failures are contained and gracefully
degraded). This is one of the core promises of the Reactive Manifesto — resilience —
and Project Reactor gives you all the tools needed to implement it directly in your
pipelines.
