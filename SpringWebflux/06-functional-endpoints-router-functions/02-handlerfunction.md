# HandlerFunction

## In Simple Terms

A `HandlerFunction<ServerResponse>` is the functional equivalent of a controller
method — it's a function that takes a `ServerRequest` and returns a
`Mono<ServerResponse>`. Handler functions contain the actual business logic that a
`RouterFunction` routes requests to.

## Simple Example

```java
@Component
public class ProductHandler {

    private final ProductService productService;

    public ProductHandler(ProductService productService) {
        this.productService = productService;
    }

    public Mono<ServerResponse> getProduct(ServerRequest request) {
        String id = request.pathVariable("id");

        return productService.getProduct(id)
            .flatMap(product -> ServerResponse.ok().bodyValue(product))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> getAllProducts(ServerRequest request) {
        return ServerResponse.ok()
            .body(productService.getAllProducts(), ProductDto.class);
    }
}
```

Each method here matches the `HandlerFunction` signature:
`Mono<ServerResponse> handle(ServerRequest request)`.

## Why It Matters

`HandlerFunction`s keep your business/HTTP-response logic as plain, easily testable
Java methods — no annotations needed, no framework "magic" translating method
signatures into HTTP semantics — everything about how the request is read and the
response is built is explicit code you can trace and test directly.
