# Validation Inside Reactive Pipelines

## In Simple Terms

Beyond simple annotations, you'll often need to weave validation right
into a reactive chain — especially when the check itself depends on
something async, like a database lookup or an external call. That means
using `.flatMap()` and `Mono.error()` to say "validate first, and only
move on if it passes."

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

If any validation step in the chain raises an error (through
`Mono.error()` inside a service call), the whole chain stops right there
and the error flows through to the controller, ready to be handled
centrally.

## Why It Matters

Writing validation as part of the reactive chain — instead of blocking to
check conditions up front — keeps your whole request handling consistently
non-blocking, which matters if you want to keep WebFlux's scalability
benefits even when validation itself needs to reach out and do I/O.
