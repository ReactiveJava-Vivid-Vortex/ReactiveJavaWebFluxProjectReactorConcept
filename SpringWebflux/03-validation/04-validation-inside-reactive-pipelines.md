# Validation Inside Reactive Pipelines

## In Simple Terms

Beyond annotation-based validation, you often need to weave validation logic
directly into a reactive chain — especially when validation depends on
asynchronous data (a database check, an external service call). This means using
operators like `.flatMap()` and `Mono.error()` to express "validate, then proceed
only if valid."

## Simple Example

```java
public Mono<OrderDto> createOrder(CreateOrderRequest request) {
    return validateInventory(request)         // async validation - checks stock levels
        .then(validateCustomer(request.customerId())) // async validation - checks customer exists
        .then(Mono.defer(() -> {
            OrderEntity entity = OrderMapper.toEntity(request);
            return orderRepository.save(entity);
        }))
        .map(OrderMapper::toDto);
}

private Mono<Void> validateInventory(CreateOrderRequest request) {
    return Flux.fromIterable(request.items())
        .flatMap(item -> inventoryService.checkStock(item.productId(), item.quantity()))
        .then(); // completes only if all items pass the check
}
```

If any validation step in the chain emits an error (via `Mono.error()` inside a
service call), the whole chain short-circuits and the error propagates to the
controller, ready to be handled centrally.

## Why It Matters

Expressing validation as part of the reactive chain (rather than blocking to check
conditions upfront) keeps your entire request-handling logic consistently
non-blocking — critical for maintaining WebFlux's scalability benefits even when
validation itself requires I/O.
