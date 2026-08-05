# Standardized Error Responses

## In Simple Terms

Beyond just using `ProblemDetail`, "standardized error responses" means designing a
**consistent overall error contract** for your entire API — the same fields, the
same structure, the same conventions — so any client integrating with any endpoint
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

Whether you adopt the RFC standard `ProblemDetail` format or a custom shape,
consistency is what actually matters most — client applications can write one
generic error-handling routine that works across your entire API, instead of custom
parsing logic per endpoint.
