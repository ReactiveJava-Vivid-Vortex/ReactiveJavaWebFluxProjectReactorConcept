# Q1. Does Bean Validation (`@Valid`) Just Work the Same as Spring MVC?

## Simple Explanation (Think of a Security Checkpoint With Two Doors)

Imagine an airport security checkpoint with two doors: Door A (a person walks
through, fully scanned before they're let in) and Door B (a conveyor belt still
delivering their bags). Validation on Door A is straightforward. Validation on
Door B needs to wait for the belt to finish arriving first.

```java
// Door A: @Valid reliably triggers
@PostMapping
public Mono<ProductDto> create(@Valid @RequestBody ProductDto dto) { ... }

// Door B: @Valid does NOT reliably auto-trigger!
@PostMapping
public Mono<ProductDto> create(@Valid @RequestBody Mono<ProductDto> dtoMono) { ... }
```

This is the single most important, easy-to-miss nuance in this whole topic.

---

## Q2. What Does Simple Field Validation Look Like?

```java
public record ProductDto(
    @NotBlank(message = "Name is required") String name,
    @Positive(message = "Price must be positive") double price
) {}

@PostMapping
public Mono<ProductDto> create(@Valid @RequestBody ProductDto dto) {
    return productService.create(dto); // @Valid triggers automatically HERE
}
```

Violations throw a `WebExchangeBindException`, typically handled centrally by a
`@ControllerAdvice` (covered in the Reactive Error Handling topic).

---

## Q3. What About Rules That Need a Database Lookup?

Simple annotations can't express "this email must not already exist" — that needs
an async check. Write a **custom validator** returning `Mono<Void>`:

```java
public Mono<Void> validate(ProductDto dto) {
    if (dto.price() <= 0) {
        return Mono.error(new ValidationException("Price must be positive"));
    }
    return repository.existsByName(dto.name())
        .flatMap(exists -> exists
            ? Mono.error(new ValidationException("Name already exists"))
            : Mono.empty());
}
```

```java
public Mono<ProductDto> createProduct(ProductDto dto) {
    return validator.validate(dto)
        .then(Mono.defer(() -> repository.save(ProductMapper.toEntity(dto))))
        .map(ProductMapper::toDto);
}
```

---

## Q4. How Do I Weave Validation Into a Larger Reactive Chain?

```java
public Mono<OrderDto> createOrder(CreateOrderRequest request) {
    return validateInventory(request)                 // async check #1
        .then(validateCustomer(request.customerId()))  // async check #2
        .then(Mono.defer(() -> orderRepository.save(OrderMapper.toEntity(request))))
        .map(OrderMapper::toDto);
}
```

If any validation step emits `Mono.error(...)`, the whole chain short-circuits and
the error propagates to the controller — ready for centralized handling.

---

## Q5. Interview-Style Q&A

### If I switch `@RequestBody ProductDto` to `@RequestBody Mono<ProductDto>`, does my existing `@Valid` still work?

**Not reliably** — this is exactly the trap from Q1. Verify with a test whenever
you use the `Mono<T>` form, and validate explicitly if it doesn't trigger.

### Can validation logic call the database?

**Yes** — but it must be expressed as a `Mono`-returning method (async), not a
synchronous Bean Validation annotation, since annotations can't await a database
call.

### What happens if I forget to call `.then()` after a validator?

The validation `Mono<Void>` is built but never subscribed to as part of the main
chain — the validation silently never runs. Always `.then()` (or `.flatMap()`) it
into the pipeline.

---

## Q6. Summary

```
Incoming request body
      │
      ▼
Simple field rules (@NotBlank, @Positive, ...)
      │  @Valid @RequestBody Dto           -> WORKS reliably
      │  @Valid @RequestBody Mono<Dto>     -> does NOT auto-trigger reliably!
      ▼
Business rules needing a DB/external check
      │  custom Mono<Void> validator, chained with .then()
      ▼
Passes all checks → proceed to service logic
Fails any check    → Mono.error(...) short-circuits → Reactive Error Handling topic
```

### One sentence to remember

> **"`@Valid` works reliably on a plain DTO body, but NOT reliably on a
> `Mono<Dto>` body — always double-check with a test whenever you use the
> Mono form."**
