# Mono/Flux Database Operations

## In Simple Terms

Every database operation through R2DBC returns a `Mono` or `Flux`, depending on
whether it can produce 0-1 or 0-N results — following exactly the same convention as
the rest of your WebFlux application, so database calls compose seamlessly into your
existing reactive pipelines.

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

Because database operations naturally return `Mono`/`Flux`, they compose directly
with the rest of your service logic using the exact same operators
(`.flatMap()`, `.map()`, `.zip()`) you use everywhere else in a reactive pipeline —
there's no special "unwrapping" needed to bridge database results into your business
logic.
