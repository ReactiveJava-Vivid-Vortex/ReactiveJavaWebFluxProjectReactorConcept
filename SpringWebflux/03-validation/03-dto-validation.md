# DTO Validation

## In Simple Terms

"DTO validation" is the practice of validating the shape and business rules of
incoming request data — annotated directly on your DTO fields — as early as possible
in the request pipeline, before that data is used to create/update entities or
trigger further logic.

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

Controller usage — `@Valid` triggers validation, and any violations throw a
`WebExchangeBindException`, which you'd typically handle via `@ControllerAdvice`
(covered in Reactive Error Handling):

```java
@PostMapping("/orders")
public Mono<OrderDto> createOrder(@Valid @RequestBody CreateOrderRequest request) {
    return orderService.createOrder(request);
}
```

## Why It Matters

Validating at the DTO layer (rather than deep inside business logic) catches
malformed requests as early as possible, with clear, standardized error responses —
preventing invalid data from ever reaching your service or repository layers in the
first place.
