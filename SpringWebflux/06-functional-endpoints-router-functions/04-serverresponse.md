# ServerResponse

## In Simple Terms

`ServerResponse` is how you build an HTTP response in the functional
model — status code, headers, body — using a fluent builder, instead of
returning a plain object (or `ResponseEntity`) from an annotated controller
method.

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

`ServerResponse`'s builder gives you the same fine-grained control over the
HTTP response (status, headers, content type, streaming body) that
`ResponseEntity` gives you in the annotation-based world — just written as
explicit method chaining, which fits naturally with the functional routing
style.
