# Standardized Error Responses

## In Simple Terms

Beyond just using `ProblemDetail`, "standardized error responses" means
designing one consistent error format for your whole API — same fields,
same structure, same conventions — so any client hitting any endpoint
knows exactly what shape to expect when something goes wrong.

## Simple Example

A consistent custom error response structure:

```java
public record ErrorResponse(
    String errorCode,
    String message,
    Instant timestamp,
    Map<String, String> details
) {
    public static ErrorResponse of(String code, String message) {
        return new ErrorResponse(code, message, Instant.now(), Map.of());
    }
}
```

Applied consistently across every exception handler:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse.of("PRODUCT_NOT_FOUND", ex.getMessage()));
    }

    @ExceptionHandler(InsufficientStockException.class)
    public ResponseEntity<ErrorResponse> handleStockError(InsufficientStockException ex) {
        return ResponseEntity.status(HttpStatus.CONFLICT)
            .body(ErrorResponse.of("INSUFFICIENT_STOCK", ex.getMessage()));
    }
}
```

## Why It Matters

Whether you go with the RFC standard `ProblemDetail` format or your own
custom shape, what really matters is consistency — client apps can write
one generic error-handling routine that works across your whole API,
instead of writing custom parsing logic for every single endpoint.
