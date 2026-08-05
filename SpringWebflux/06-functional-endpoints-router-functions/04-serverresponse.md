# ServerResponse

## In Simple Terms

`ServerResponse` is the functional model's way of building an HTTP response — status
code, headers, and body — using a fluent builder API, instead of returning a plain
object (or `ResponseEntity`) from an annotated controller method.

## Simple Example

```java
// 200 OK with a body
ServerResponse.ok().bodyValue(product);

// 201 Created with a body
ServerResponse.status(HttpStatus.CREATED).bodyValue(createdProduct);

// 404 Not Found, no body
ServerResponse.notFound().build();

// 204 No Content
ServerResponse.noContent().build();

// Streaming a Flux as the body
ServerResponse.ok()
    .contentType(MediaType.APPLICATION_NDJSON)
    .body(productFlux, ProductDto.class);

// Custom headers
ServerResponse.ok()
    .header("X-Total-Count", String.valueOf(totalCount))
    .bodyValue(products);
```

## Why It Matters

`ServerResponse`'s builder API gives you the same fine-grained control over the HTTP
response (status, headers, content type, streaming body) as `ResponseEntity` does in
the annotation-based model — just expressed through explicit method chaining rather
than annotations, fitting naturally with the functional routing style.
