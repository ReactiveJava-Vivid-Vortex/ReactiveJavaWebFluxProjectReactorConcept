# Mono/Flux Database Operations

## In Simple Terms

Every database call through R2DBC returns a `Mono` or `Flux`, depending on
whether it can give back 0-1 or 0-N results — the exact same convention as
the rest of your WebFlux app, so database calls fit right into your
existing reactive pipelines without any special handling.

## Simple Example

```java
// findById() -> Mono (0 or 1 result)
Mono<ProductEntity> product = productRepository.findById("P123");

// findAll() -> Flux (0 to N results)
Flux<ProductEntity> allProducts = productRepository.findAll();

// Custom query returning a single count -> Mono<Long>
Mono<Long> totalCount = productRepository.count();

// Composing multiple database calls reactively
public Mono<OrderSummary> getOrderSummary(String orderId) {
    return orderRepository.findById(orderId)
        .flatMap(order -> customerRepository.findById(order.getCustomerId())
            .map(customer -> new OrderSummary(order, customer)));
}
```

## Why It Matters

Because database calls come back as `Mono`/`Flux` naturally, they slot
right into your service logic with the exact same operators
(`.flatMap()`, `.map()`, `.zip()`) you use everywhere else in a reactive
pipeline — there's no special "unwrapping" step needed to bring database
results into your business logic.
