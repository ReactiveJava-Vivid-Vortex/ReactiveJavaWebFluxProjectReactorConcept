# Downstream Service Failures

## In Simple Terms

In a microservices architecture, any service you call over the network can fail —
timeouts, 500 errors, connection refused. A resilient reactive service needs a
deliberate strategy for each downstream call: what to do when it fails, so the
failure doesn't cascade into your own service failing entirely.

## Simple Example

```java
public Mono<ProductDetails> getProductDetails(String id) {
    return productServiceClient.getProduct(id)
        .timeout(Duration.ofSeconds(2))
        .retryWhen(Retry.backoff(2, Duration.ofMillis(200))
            .filter(e -> e instanceof WebClientResponseException.ServiceUnavailable))
        .onErrorResume(error -> {
            log.error("Product service call failed for {}: {}", id, error.getMessage());
            return Mono.error(new DownstreamServiceException("Product service unavailable"));
        });
}
```

This pipeline: bounds the wait time, retries transient `503` errors with backoff, and
converts any remaining failure into a clear, well-logged domain exception rather than
letting a raw `WebClientResponseException` propagate uncontrolled.

## Why It Matters

Explicitly handling downstream failures at every service-to-service call boundary is
what prevents a single failing dependency from cascading into a full outage —
exactly the "Resilient" trait from the Reactive Manifesto ([[resilient]]), applied
concretely to real microservice communication.
