# Problem Details (RFC 7807 / RFC 9457)

## In Simple Terms

**Problem Details** is a standardized JSON format (defined by RFC 7807, updated by
RFC 9457) for representing HTTP API errors consistently — instead of every API
inventing its own ad-hoc error JSON shape. Spring provides built-in support via
`ProblemDetail`, which WebFlux controllers can return directly.

## Simple Example

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ProductNotFoundException.class)
    public ProblemDetail handleNotFound(ProductNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(
            HttpStatus.NOT_FOUND, ex.getMessage()
        );
        problem.setTitle("Product Not Found");
        problem.setProperty("productId", ex.getProductId());
        return problem;
    }
}
```

The resulting JSON response follows the standard shape:

```json
{
  "type": "about:blank",
  "title": "Product Not Found",
  "status": 404,
  "detail": "Product not found with id: abc123",
  "productId": "abc123"
}
```

## Why It Matters

Using the standardized Problem Details format means API clients (and tooling) can
rely on a consistent, well-known error shape across different services and teams —
instead of every API having its own bespoke error JSON structure that clients need to
learn individually.
