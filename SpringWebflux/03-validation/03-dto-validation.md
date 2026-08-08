# DTO Validation

## In Simple Terms

"DTO validation" means checking the shape and rules of incoming request
data — right there on the DTO's fields — as early as possible in the
request, before that data ever gets used to create or update anything.

## Simple Example

```java
public record CreateOrderRequest(
    @NotBlank String customerId,
    @NotEmpty List<@Valid OrderItemRequest> items,
    @Positive double totalAmount
) {}

public record OrderItemRequest(
    @NotBlank String productId,
    @Min(1) int quantity
) {}
```

Controller usage — `@Valid` triggers the checks, and any violations throw a
`WebExchangeBindException`, which you'd typically handle with
`@ControllerAdvice` (covered under Reactive Error Handling):

```java
@PostMapping("/orders")
public Mono<OrderDto> createOrder(@Valid @RequestBody CreateOrderRequest request) {
    return orderService.createOrder(request);
}
```

## Why It Matters

Validating right at the DTO layer (instead of deep inside your business
logic) catches bad requests as early as possible, with clear, consistent
error responses — stopping invalid data from ever reaching your service or
repository layers in the first place.
