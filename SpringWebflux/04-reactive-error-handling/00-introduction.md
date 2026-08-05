# Reactive Error Handling — Topic Overview

## What Is This Topic About? (In Simple Terms)

This topic applies Project Reactor's error-handling operators
(`Mono.error()`, `switchIfEmpty()`) specifically to a Spring WebFlux application's
needs: turning failures and "not found" results into clean, consistent, predictable
HTTP responses — instead of leaking raw exceptions or ambiguous empty responses to
API clients.

The recommended flow: define **custom exceptions** for meaningful domain failures,
signal them from your service layer with `Mono.error()`/`switchIfEmpty()`, and catch
them **centrally** in a `@RestControllerAdvice`, which maps each exception type to
the right HTTP status:

```java
// Service layer — signal a specific failure
public Mono<ProductDto> getProduct(String id) {
    return repository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
        .map(ProductMapper::toDto);
}

// Centralized handler — every controller benefits automatically
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse.of("PRODUCT_NOT_FOUND", ex.getMessage()));
    }
}
```

The last piece is **response shape consistency** — whether you adopt the RFC
standard `ProblemDetail` format or your own custom `ErrorResponse` record, every
error your API returns should follow the *same* structure, so client applications
can write one generic error-parsing routine instead of custom logic per endpoint.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Custom Exceptions** | Domain-specific exception classes (`ProductNotFoundException`) instead of generic `RuntimeException`. |
| 2 | **Mono.error()** | The reactive way to signal failure from a service method — a returned value, not a thrown exception. |
| 3 | **switchIfEmpty()** | Standard pattern to turn "nothing found" into an explicit error (or a fallback value). |
| 4 | **Exception Factory** | A helper class centralizing exception creation/messages, avoiding duplicated `new SomeException(...)` calls. |
| 5 | **Controller Advice** | `@RestControllerAdvice` + `@ExceptionHandler` — centralized, consistent exception → HTTP response mapping. |
| 6 | **Problem Details (RFC 7807/9457)** | Spring's built-in `ProblemDetail` — a standardized JSON error format instead of an ad-hoc one. |
| 7 | **Standardized error responses** | Whatever shape you pick, keep it consistent across your entire API surface. |

## How It All Fits Together

```
Service layer detects a failure
      │
      ▼
Mono.error(customException)  /  switchIfEmpty(Mono.error(customException))
      │
      ▼
Error propagates up through the reactive chain (controller does nothing special)
      │
      ▼
@RestControllerAdvice's matching @ExceptionHandler catches it
      │
      ▼
Consistent, standardized JSON error response (ProblemDetail or custom ErrorResponse)
```

Think of this topic as building one centralized "translation layer": domain
exceptions go in, consistent HTTP error responses come out — every controller
benefits without repeating the mapping logic itself.
