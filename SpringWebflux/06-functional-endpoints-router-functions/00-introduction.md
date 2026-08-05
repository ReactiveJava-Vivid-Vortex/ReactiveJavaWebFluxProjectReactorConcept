# Functional Endpoints (Router Functions) — Topic Overview

## What Is This Topic About? (In Simple Terms)

Instead of scattering `@GetMapping`/`@PostMapping` annotations across controller
classes, WebFlux offers a completely different, **functional** style of declaring
routes — many production teams actually prefer this for its explicitness. You
declare a `RouterFunction` that maps URL patterns to plain `HandlerFunction`
methods, all visible in one place:

```java
@Bean
public RouterFunction<ServerResponse> productRoutes(ProductHandler handler) {
    return RouterFunctions.route()
        .GET("/products", handler::getAllProducts)
        .GET("/products/{id}", handler::getProduct)
        .POST("/products", handler::createProduct)
        .build();
}
```

A `HandlerFunction` is just a method with the signature
`Mono<ServerResponse> handle(ServerRequest request)` — no annotations at all. You
read request data explicitly (`request.pathVariable("id")`,
`request.bodyToMono(Dto.class)`) and build the response explicitly
(`ServerResponse.ok().bodyValue(...)`):

```java
public Mono<ServerResponse> getProduct(ServerRequest request) {
    String id = request.pathVariable("id");
    return productService.getProduct(id)
        .flatMap(product -> ServerResponse.ok().bodyValue(product))
        .switchIfEmpty(ServerResponse.notFound().build());
}
```

**Important gotcha:** since there's no `@Valid` annotation to auto-trigger, both
validation and exception handling must be done **explicitly inside the handler
function** — using a `Validator` bean directly, and `onErrorResume()` in the
reactive chain (or a registered `WebExceptionHandler` for global handling).

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **RouterFunction** | Explicit, centralized route declarations (URL + method → handler) — the functional alternative to `@GetMapping`. |
| 2 | **HandlerFunction** | A plain method: `Mono<ServerResponse> handle(ServerRequest)` — contains the actual logic, no annotations. |
| 3 | **ServerRequest** | Explicit access to path variables, query params, headers, and body — no annotation injection. |
| 4 | **ServerResponse** | Fluent builder for status/headers/body (e.g. `ServerResponse.ok().bodyValue(...)`) — like `ResponseEntity`. |
| 5 | **Functional Routing** | Composing multiple `RouterFunction`s together with `.and()` for a modular, centralized routing table. |
| 6 | **Functional Validation** | No auto `@Valid` — validate explicitly inside the handler using a `Validator` bean. |
| 7 | **Functional Exception Handling** | Handle errors locally with `onErrorResume()` in the handler, or globally via a custom `WebExceptionHandler`. |

## How It All Fits Together

```
RouterFunction (declared centrally, e.g. in @Configuration)
      │  maps  GET /products/{id}  ──────▶  ProductHandler::getProduct
      ▼
HandlerFunction: Mono<ServerResponse> getProduct(ServerRequest request)
      │
      ├── request.pathVariable("id")   ← read request data explicitly
      ├── (explicit validation here, if needed)
      ├── call productService...
      └── ServerResponse.ok().bodyValue(...)   ← build response explicitly
```

Both styles (annotation-based and functional) produce the exact same runtime
behavior — the choice is purely about whether your team prefers annotations
(implicit, less code) or explicit function composition (more visible, more
testable as plain Java methods).
