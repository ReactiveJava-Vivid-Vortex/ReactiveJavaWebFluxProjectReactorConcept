# Functional Exception Handling

## In Simple Terms

In the functional model, `@ControllerAdvice`/`@ExceptionHandler` still
work globally, but you can also handle errors locally, right inside a
handler function's chain, using the same `onErrorResume()`/`onErrorReturn()`
operators you'd use anywhere else in Project Reactor.

## Simple Example

```java
public Mono<ServerResponse> getProduct(ServerRequest request) {
    String id = request.pathVariable("id");

    return productService.getProduct(id)
        .flatMap(product -> ServerResponse.ok().bodyValue(product))
        .onErrorResume(ProductNotFoundException.class, e ->
            ServerResponse.status(HttpStatus.NOT_FOUND)
                .bodyValue(new ErrorResponse("PRODUCT_NOT_FOUND", e.getMessage()))
        )
        .onErrorResume(Exception.class, e ->
            ServerResponse.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .bodyValue(new ErrorResponse("INTERNAL_ERROR", "Something went wrong"))
        );
}
```

For truly global handling across all functional routes, Spring also lets
you register a custom `WebExceptionHandler` bean, which acts a lot like
`@ControllerAdvice` but fits the functional style.

## Why It Matters

Handling errors right there in the handler's own chain keeps the
error-handling logic close to the operation it applies to — useful when
different endpoints need meaningfully different responses for the same
exception, something a single global `@ControllerAdvice` handler can't
easily express.
