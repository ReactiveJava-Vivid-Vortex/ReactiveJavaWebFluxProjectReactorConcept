# Problem Details (RFC 7807 / RFC 9457)

## In Simple Terms

Problem Details is a standardized JSON shape (defined by RFC 7807, updated
by RFC 9457) for representing API errors consistently — instead of every
API inventing its own custom error format. Spring supports it natively
through `ProblemDetail`, which WebFlux controllers can return directly.

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

The resulting JSON follows the standard shape:

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

Using the standard Problem Details format means API clients (and tooling)
can count on one familiar error shape across different services and
teams — instead of every API having its own bespoke error JSON that
clients need to learn one by one.
