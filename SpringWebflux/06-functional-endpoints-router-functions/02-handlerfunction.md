# HandlerFunction

## In Simple Terms

A `HandlerFunction<ServerResponse>` is the functional equivalent of a
controller method — a plain function that takes a `ServerRequest` and
hands back a `Mono<ServerResponse>`. Handler functions hold the actual
business logic that a `RouterFunction` routes requests to.

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

Each method here matches the `HandlerFunction` shape:
`Mono<ServerResponse> handle(ServerRequest request)`.

## Why It Matters

`HandlerFunction`s keep your logic as plain, easy-to-test Java methods —
no annotations needed, no framework "magic" translating a method signature
into HTTP behavior — everything about reading the request and building the
response is code you can trace and test directly.
