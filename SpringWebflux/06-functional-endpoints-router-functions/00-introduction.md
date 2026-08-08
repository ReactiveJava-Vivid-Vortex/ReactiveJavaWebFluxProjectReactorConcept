# Q1. What Are Functional Endpoints, and Why Would I Use Them Instead of Annotations?

## Simple Explanation (Think of a Restaurant Menu vs a Chef Taking Verbal Orders)

`@GetMapping`/`@PostMapping` annotations are like a chef who reacts to whatever
orders come in, wherever they're scribbled (scattered across many controller
classes). Functional routing is a **printed menu** — every route is listed
explicitly, in one place, mapped directly to what makes it:

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

Both styles produce **identical runtime behavior** — this is purely about whether
your team prefers implicit annotations or explicit function composition.

---

## Q2. What Does a Handler Function Look Like?

A `HandlerFunction` is just a plain method: `Mono<ServerResponse> handle(ServerRequest)`
— no annotations at all.

```java
public Mono<ServerResponse> getProduct(ServerRequest request) {
    String id = request.pathVariable("id");           // read data EXPLICITLY
    return productService.getProduct(id)
        .flatMap(product -> ServerResponse.ok().bodyValue(product))  // build response EXPLICITLY
        .switchIfEmpty(ServerResponse.notFound().build());
}
```

---

## Q3. `ServerRequest` and `ServerResponse` — The Explicit Read/Write API

```java
// ServerRequest — reading
String id = request.pathVariable("id");
Optional<String> category = request.queryParam("category");
Mono<ProductDto> body = request.bodyToMono(ProductDto.class);

// ServerResponse — writing
ServerResponse.ok().bodyValue(product);                        // 200 OK
ServerResponse.status(HttpStatus.CREATED).bodyValue(created);   // 201 Created
ServerResponse.notFound().build();                              // 404, no body
```

---

## Q4. How Do I Compose Multiple Route Groups Together?

```java
@Bean
public RouterFunction<ServerResponse> allRoutes(ProductHandler p, OrderHandler o) {
    RouterFunction<ServerResponse> productRoutes = RouterFunctions.route()
        .GET("/products", p::getAll).GET("/products/{id}", p::getById).build();

    RouterFunction<ServerResponse> orderRoutes = RouterFunctions.route()
        .GET("/orders", o::getAll).POST("/orders", o::create).build();

    return productRoutes.and(orderRoutes); // compose independently-defined groups
}
```

---

## Q5. What Happens to `@Valid` in Functional Endpoints?

**It doesn't exist here — there's no annotation to auto-trigger.** Validation and
error handling must be done **explicitly, inside the handler function:**

```java
public Mono<ServerResponse> createProduct(ServerRequest request) {
    return request.bodyToMono(ProductDto.class)
        .flatMap(dto -> {
            Errors errors = new BeanPropertyBindingResult(dto, "productDto");
            validator.validate(dto, errors);
            if (errors.hasErrors()) {
                return ServerResponse.badRequest().bodyValue(new ErrorResponse(errors.getAllErrors().get(0).getDefaultMessage()));
            }
            return productService.create(dto).flatMap(created -> ServerResponse.status(HttpStatus.CREATED).bodyValue(created));
        })
        .onErrorResume(ProductNotFoundException.class, e -> ServerResponse.status(404).bodyValue(new ErrorResponse(e.getMessage())));
}
```

---

## Q6. Interview-Style Q&A

### Can I mix functional and annotated endpoints in the same application?

**Yes.** Nothing stops you — both are registered by the same underlying WebFlux
infrastructure, side by side, with no conflict.

### Does `@ControllerAdvice` still work with functional endpoints?

Not automatically the same way — you handle errors locally with `.onErrorResume()`
in the handler, or register a global `WebExceptionHandler` bean for
functional-style centralized handling.

### Is one style faster than the other at runtime?

**No** — they compile down to the same underlying routing/handling machinery.
Choice is purely stylistic/organizational.

---

## Q7. Summary

```
RouterFunction (declared centrally, e.g. in @Configuration)
      │  maps  GET /products/{id}  ──────▶  ProductHandler::getProduct
      ▼
HandlerFunction: Mono<ServerResponse> getProduct(ServerRequest request)
      │
      ├── request.pathVariable("id")   ← read request data explicitly
      ├── (explicit validation, if needed — no @Valid here)
      └── ServerResponse.ok().bodyValue(...)   ← build response explicitly
```

### One sentence to remember

> **"Functional endpoints trade annotation 'magic' for explicit, centralized,
> plain-Java-method routing — same runtime behavior, more visible code, no
> auto-validation."**
