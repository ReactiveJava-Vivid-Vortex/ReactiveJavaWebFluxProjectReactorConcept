# Resilience (Real-World Patterns)

## In Simple Terms

Building on the Reactive Manifesto's "Resilient" idea ([[resilient]]),
real microservices usually stack several patterns together: timeouts,
retries with a growing delay, fallbacks, and sometimes circuit breakers
(via libraries like Resilience4j) — layered up to handle the many
different ways a downstream call can go wrong.

## Simple Example

A layered resilience strategy combining multiple patterns:

```java
public Mono<InventoryStatus> checkInventory(String productId) {
    return inventoryServiceClient.checkStock(productId)
        .timeout(Duration.ofSeconds(1))                         // bound the wait
        .retryWhen(Retry.backoff(2, Duration.ofMillis(100))      // retry transient errors
            .filter(e -> e instanceof TimeoutException))
        .transform(CircuitBreakerOperator.of(circuitBreaker))    // trip after repeated failures
        .onErrorResume(error -> {                                 // final fallback
            log.warn("Inventory check failed, assuming available", error);
            return Mono.just(InventoryStatus.assumeAvailable());
        });
}
```

The circuit breaker (from Resilience4j's reactive support) stops you from
repeatedly hitting a service that's clearly down, "failing fast" instead
of continuing to send requests (and burning resources) at something known
to be unhealthy.

## Why It Matters

Layering these patterns together — instead of relying on just one — gives
you defense in depth against the many ways distributed systems actually
fail: slow responses (timeout), brief hiccups (retry), sustained outages
(circuit breaker), and total unavailability (fallback).
