# ServerRequest

## In Simple Terms

`ServerRequest` represents the incoming HTTP request in the functional
WebFlux model — giving you access to path variables, query parameters,
headers, and the request body, all through direct method calls instead of
annotation-driven injection (`@PathVariable`, `@RequestParam`, etc.).

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

Getting request data through direct method calls (instead of annotation
injection) makes handler functions easy to unit test — you can build a mock
`ServerRequest` right in a test and call your handler with it, without
needing a full Spring MVC test context.
