# Functional Validation

## In Simple Terms

In the functional WebFlux model, since there's no `@Valid` annotation on a method
parameter to trigger automatically, validation is performed **explicitly** inside
your handler function — typically using a `Validator` bean or custom validation
logic applied directly to the parsed request body.

## Simple Example

```java
@Component
public class ProductHandler {

    private final Validator validator;
    private final ProductService productService;

    public Mono<ServerResponse> createProduct(ServerRequest request) {
        return request.bodyToMono(ProductDto.class)
            .flatMap(dto -> {
                Errors errors = new BeanPropertyBindingResult(dto, "productDto");
                validator.validate(dto, errors);

                if (errors.hasErrors()) {
                    String message = errors.getAllErrors().get(0).getDefaultMessage();
                    return ServerResponse.badRequest().bodyValue(new ErrorResponse(message));
                }

                return productService.create(dto)
                    .flatMap(created -> ServerResponse.status(HttpStatus.CREATED).bodyValue(created));
            });
    }
}
```

## Why It Matters

Because functional endpoints don't get automatic annotation-driven validation, being
explicit about validation inside each handler is essential — it's slightly more
verbose than `@Valid`, but it makes validation logic fully visible and controllable
directly in your handler code, with no hidden "magic" behavior to account for.
