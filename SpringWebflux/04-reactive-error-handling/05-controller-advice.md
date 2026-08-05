# Controller Advice

## In Simple Terms

`@ControllerAdvice` (combined with `@ExceptionHandler`) lets you handle exceptions
**centrally**, across all controllers, translating each exception type into a
consistent, appropriate HTTP response — instead of repeating try/catch-like logic
(`onErrorResume`) in every single controller method.

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

With this in place, controllers can stay clean and focused — errors bubbling up
from a `Mono.error()` anywhere in the chain are automatically caught and translated
by the matching `@ExceptionHandler`:

```java
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productService.getProduct(id); // errors handled globally, not here
}
```

## Why It Matters

Centralized exception handling ensures consistent, well-structured error responses
across your entire API, avoids duplicated error-handling code in every controller,
and makes it easy to add new exception types (and their corresponding HTTP status
codes) in one place as your application grows.
