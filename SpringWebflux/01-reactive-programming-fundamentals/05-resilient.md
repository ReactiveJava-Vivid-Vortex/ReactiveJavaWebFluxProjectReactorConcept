# Resilient

## In Simple Terms

"Resilient" means a system keeps working even when parts of it break — one
failing dependency shouldn't drag the whole thing down with it. Resilience
comes from keeping failures contained to one spot, and recovering
gracefully (fallbacks, retries, and so on).

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

Even if the recommendation service goes completely down, the rest of the
app (and this endpoint specifically) keeps working — just with a generic
fallback instead of personalized recommendations.

## Why It Matters

In a system made up of many interdependent services, resilience is what
stops one failing service from taking the whole thing down with it (a
"cascading failure"). Tools like `onErrorResume`, `retry`, and `timeout`
give you the building blocks to design resilience directly into how your
services talk to each other.
