# Q1. How Should Failures Turn Into HTTP Responses?

## Simple Explanation (Think of a Hospital's Triage Desk, Not Every Doctor Deciding Alone)

Without central error handling, every controller method has to individually
decide "what HTTP status should THIS failure produce?" — inconsistent, duplicated,
error-prone. A **triage desk** (`@RestControllerAdvice`) instead looks at every
incoming problem once, and routes it to the right response, consistently, no
matter which "doctor" (controller) it came from.

```java
// Every controller just signals failure — doesn't decide the HTTP response itself
public Mono<ProductDto> getProduct(String id) {
    return repository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
        .map(ProductMapper::toDto);
}

// ONE triage desk decides the response for every ProductNotFoundException, anywhere
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ProductNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ProductNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse.of("PRODUCT_NOT_FOUND", ex.getMessage()));
    }
}
```

---

## Q2. Why Custom Exceptions Instead of Generic `RuntimeException`?

```java
public class ProductNotFoundException extends RuntimeException {
    private final String productId;
    public ProductNotFoundException(String id) {
        super("Product not found: " + id);
        this.productId = id;
    }
}
```

Generic exceptions give your `@ControllerAdvice` nothing specific to match
against. Named, domain-specific exceptions let each one map to exactly the right
status code and message.

---

## Q3. `Mono.error()` and `switchIfEmpty()` — The Two Signaling Tools

```java
// Mono.error(): the reactive equivalent of "throw" — a returned VALUE, not a thrown exception
public Mono<ProductDto> getProduct(String id) {
    if (id == null) return Mono.error(new IllegalArgumentException("id required"));
    return repository.findById(id)
        .switchIfEmpty(Mono.error(new ProductNotFoundException(id))) // empty -> explicit error
        .map(ProductMapper::toDto);
}
```

`switchIfEmpty()` is the standard pattern for turning "nothing found" into either
an explicit error OR a fallback value — see the Error Handling topic in the
Project Reactor notes for the full operator toolkit.

---

## Q4. What's the Standard Response Format?

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ProductNotFoundException.class)
    public ProblemDetail handleNotFound(ProductNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Product Not Found");
        return problem;
    }
}
```

Response (standard RFC 7807/9457 shape):
```json
{ "type": "about:blank", "title": "Product Not Found", "status": 404, "detail": "Product not found: abc123" }
```

Whether you use `ProblemDetail` or your own custom `ErrorResponse` record, **the
one rule that matters most is consistency** — every error in your API should look
the same shape, so clients can write one generic parsing routine.

---

## Q5. Interview-Style Q&A

### Does throwing a plain `throw new SomeException()` inside a `.map()` lambda work the same as `Mono.error()`?

Usually yes for synchronous operators — Reactor catches synchronous exceptions and
converts them into `onError()` signals — but explicitly returning `Mono.error()`
is clearer and safer, especially in asynchronous contexts.

### Do I need a try/catch in my controller for this to work?

**No** — the error propagates as an `onError()` signal through the reactive chain
automatically; `@ControllerAdvice` intercepts it without any manual try/catch.

### What HTTP status should an unexpected, unhandled exception return?

Register a catch-all `@ExceptionHandler(Exception.class)` returning `500 Internal
Server Error` — never let a raw stack trace leak to the client.

---

## Q6. Summary

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

### One sentence to remember

> **"Signal failures with Mono.error()/custom exceptions, and let ONE
> centralized @RestControllerAdvice translate every exception type into a
> consistent HTTP response — never decide the status code in each controller."**
