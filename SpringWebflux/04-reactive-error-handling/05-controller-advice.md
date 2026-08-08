# Controller Advice

## In Simple Terms

`@ControllerAdvice` (paired with `@ExceptionHandler`) lets you handle
exceptions in one central place, across every controller, turning each
exception type into a consistent, appropriate HTTP response — instead of
repeating try/catch-style logic in every single controller method.

## Simple Example

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("PRODUCT_NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleBadRequest(IllegalArgumentException ex) {
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("INVALID_INPUT", ex.getMessage()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        return ResponseEntity.internalServerError()
            .body(new ErrorResponse("INTERNAL_ERROR", "Something went wrong"));
    }
}
```

With this set up, controllers stay clean and focused — errors bubbling up
from a `Mono.error()` anywhere in the chain get automatically caught and
translated by the matching `@ExceptionHandler`:

```java
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productService.getProduct(id); // errors handled globally, not here
}
```

## Why It Matters

Handling exceptions in one central place keeps your whole API's error
responses consistent and well-structured, avoids repeating error-handling
code across every controller, and makes adding a new exception type (and
its matching status code) a one-place change as your app grows.
