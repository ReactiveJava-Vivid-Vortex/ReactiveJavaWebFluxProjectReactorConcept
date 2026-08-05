# Resilient

## In Simple Terms

"Resilient" means a system stays responsive even when **parts of it fail** — a
single failing dependency shouldn't cascade into a total outage. Resilience is
achieved through isolation (failures contained to one component) and graceful
recovery (fallbacks, retries, circuit breaking).

## Simple Example

```java
public Mono<Recommendations> getRecommendations(String userId) {
    return recommendationService.getFor(userId)
        .timeout(Duration.ofMillis(500))
        .retryWhen(Retry.backoff(2, Duration.ofMillis(100)))
        .onErrorResume(error -> {
            log.warn("Recommendation service failed, using generic fallback", error);
            return Mono.just(Recommendations.generic()); // system stays functional
        });
}
```

Even if the recommendation service is completely down, the rest of the
application (and this endpoint specifically) continues to function — just with a
generic fallback instead of personalized recommendations.

## Why It Matters

In a microservices architecture with many interdependent services, resilience is
what prevents a single failing service from taking down the entire system (a
"cascading failure"). Reactive error-handling operators (`onErrorResume`, `retry`,
`timeout`) give you the building blocks to design resilience directly into your
service composition.
