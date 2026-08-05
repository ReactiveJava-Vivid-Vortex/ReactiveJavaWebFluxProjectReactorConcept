# Resilience (Real-World Patterns)

## In Simple Terms

Building on the Reactive Manifesto's "Resilient" principle ([[resilient]]), real-world
microservices combine several concrete patterns together: timeouts, retries with
backoff, fallbacks, and sometimes circuit breakers (via libraries like Resilience4j)
— layered together to handle the many different ways a downstream call can fail.

## Simple Example

A comprehensive resilience strategy combining multiple patterns:

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

The circuit breaker (from Resilience4j's reactive integration) prevents repeatedly
calling a service that's clearly down, "failing fast" instead of continuing to send
requests (and consuming resources) to a service that's known to be unhealthy.

## Why It Matters

Layering these patterns together — rather than relying on just one — provides
defense in depth against the many different failure modes real distributed systems
experience: slow responses (timeout), transient blips (retry), sustained outages
(circuit breaker), and complete unavailability (fallback).
