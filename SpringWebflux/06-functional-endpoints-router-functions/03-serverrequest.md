# ServerRequest

## In Simple Terms

`ServerRequest` represents the incoming HTTP request in the functional WebFlux
model — giving you access to path variables, query parameters, headers, and the
request body, all through explicit method calls rather than annotation-driven
injection (`@PathVariable`, `@RequestParam`, etc.).

## Simple Example

```java
public Mono<ServerResponse> handleRequest(ServerRequest request) {
    String id = request.pathVariable("id");                          // path variable
    Optional<String> category = request.queryParam("category");      // query parameter
    String contentType = request.headers().firstHeader("Content-Type"); // header

    Mono<ProductDto> bodyMono = request.bodyToMono(ProductDto.class); // request body

    return bodyMono.flatMap(dto -> productService.create(dto))
        .flatMap(created -> ServerResponse.status(HttpStatus.CREATED).bodyValue(created));
}
```

## Why It Matters

`ServerRequest`'s explicit, method-call-based access to request data (rather than
annotation injection) makes handler functions straightforward to unit test — you can
construct a mock `ServerRequest` directly in a test and call your handler method with
it, without needing a full Spring MVC test context.
