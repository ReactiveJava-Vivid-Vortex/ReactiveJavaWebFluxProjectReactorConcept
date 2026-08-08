# Functional Validation

## In Simple Terms

In the functional WebFlux model, there's no `@Valid` annotation on a
method parameter to trigger checks automatically, so validation happens
explicitly inside your handler function — usually with a `Validator` bean
or custom logic applied directly to the parsed request body.

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

Since functional endpoints don't get automatic validation from
annotations, being explicit about it in each handler is a must — it's a
bit more code than `@Valid`, but it makes validation fully visible right
there in your handler, with no hidden framework behavior to keep track of.
