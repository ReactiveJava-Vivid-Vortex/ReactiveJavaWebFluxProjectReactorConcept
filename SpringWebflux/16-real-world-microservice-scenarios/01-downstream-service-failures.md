# Downstream Service Failures

## In Simple Terms

In a microservices setup, any service you call over the network can fail
— timeouts, 500 errors, connection refused. A resilient service needs a
real plan for each downstream call: what happens when it fails, so that
failure doesn't ripple out and take your own service down with it.

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

This pipeline puts a limit on the wait, retries brief `503` errors with a
growing delay, and turns whatever's left into a clear, well-logged
domain-specific error instead of letting a raw `WebClientResponseException`
leak out uncontrolled.

## Why It Matters

Explicitly handling failures at every service-to-service boundary is what
stops one failing dependency from turning into a full outage — exactly the
"Resilient" trait from the Reactive Manifesto ([[resilient]]), put into
practice for real microservice communication.
