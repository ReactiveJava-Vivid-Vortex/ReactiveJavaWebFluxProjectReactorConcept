# Q1. What Is This Bonus Topic For?

## Simple Explanation (Think of the Fine Print Nobody Reads Until It Bites Them)

Every framework has a handful of "wait, that's not what I assumed!" moments that
even experienced Project Reactor users hit the first time they build a real
WebFlux app. This topic collects them — Spring-specific integration quirks that
aren't really about `Mono`/`Flux` themselves, but about how WebFlux wires them into
HTTP.

```
The #1 most dangerous one: .block() COMPILES FINE, but using it in a
controller/filter can throw at runtime — and even when it doesn't throw,
it silently defeats the entire point of using WebFlux.
```

---

## Q2. "Why Did My 404 Turn Into a 200 With an Empty Body?"

```java
// Returning an empty Mono does NOT automatically become 404!
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
    // empty result -> client gets 200 OK, empty body — NOT 404!
}
```

Fix: be explicit.

```java
@GetMapping("/products/{id}")
public Mono<ResponseEntity<ProductDto>> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto)
        .map(ResponseEntity::ok)
        .defaultIfEmpty(ResponseEntity.notFound().build()); // NOW explicit
}
```

---

## Q3. "Can I Just Call `.block()` to Get the Value Right Now?"

**Never inside request-handling code.**

```java
// BAD — throws IllegalStateException in many cases; even when it doesn't,
// it freezes a precious event-loop thread
@GetMapping("/products/{id}")
public ProductDto getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto).block();
}

// GOOD — return the Mono, let WebFlux subscribe non-blockingly
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return productRepository.findById(id).map(ProductMapper::toDto);
}
```

`.block()` is only acceptable in genuinely non-reactive contexts entirely outside
the request pipeline — a one-off script's `main()`, or test setup.

---

## Q4. "Does `@RequestBody Mono<T>` Behave Exactly Like `@RequestBody T`?"

**Not quite.** Plain `T` reliably triggers `@Valid`; `Mono<T>` does **not**
reliably auto-trigger it.

```java
@PostMapping public Mono<OrderDto> create(@Valid @RequestBody OrderDto dto) { ... }        // @Valid WORKS
@PostMapping public Mono<OrderDto> create(@Valid @RequestBody Mono<OrderDto> dtoMono) { ... } // @Valid unreliable!
```

Default to plain `T` unless you have a specific reason to use `Mono<T>` — and
validate explicitly inside `.flatMap()` if you do.

---

## Q5. "Do I Have to Pick Either Annotated OR Functional Endpoints?"

**No** — they coexist freely in the same application:

```java
@RestController @RequestMapping("/products")
public class ProductController { ... }   // annotated style

@Bean
public RouterFunction<ServerResponse> orderRoutes(OrderHandler handler) { ... } // functional style
```

Both are registered by the same underlying infrastructure — useful for
incremental migrations, one resource at a time.

---

## Q6. "Does WebFlux Require 100% Non-Blocking Code, Zero Exceptions?"

**No** — a common over-correction. If you genuinely have no non-blocking
alternative (a legacy SDK), you can isolate it correctly:

```java
Mono.fromCallable(this::callLegacySdk)
    .subscribeOn(Schedulers.boundedElastic()) // isolates the blocking call
    .map(ResultMapper::toDto);
```

This endpoint is less scalable than a fully non-blocking one, but it's **correct**
— the real rule is narrower: never let a blocking call land on an event-loop
thread, not "never use anything blocking, ever, anywhere."

---

## Q7. Interview-Style Q&A

### If my controller returns `Mono.empty()`, what status code does the client see by default?

`200 OK` with an empty body — not `404`. You must opt into 404 explicitly.

### What's the actual runtime consequence of calling `.block()` in a WebFlux controller?

Best case: it "just" freezes an event-loop thread, degrading concurrent
throughput. Worst case: Reactor detects it and throws `IllegalStateException`
immediately.

### Is it acceptable to have ANY blocking call in a "real" WebFlux application?

**Yes** — as long as it's correctly isolated on `Schedulers.boundedElastic()`. A
few well-isolated blocking calls don't disqualify an app from being genuinely
reactive.

---

## Q8. Summary

| # | Nuance | One-Line Fix |
|---|---|---|
| 1 | Empty Mono ≠ automatic 404 | `.defaultIfEmpty(ResponseEntity.notFound()...)` explicitly |
| 2 | Never call `.block()` on WebFlux threads | Return the Mono/Flux instead |
| 3 | `@RequestBody T` vs `Mono<T>` | Plain `T` for reliable `@Valid`; validate manually for `Mono<T>` |
| 4 | Functional & annotated endpoints coexist | Mix freely, migrate incrementally |
| 5 | Isolating unavoidable blocking calls | `.subscribeOn(Schedulers.boundedElastic())`, not avoidance |

### One sentence to remember

> **"WebFlux's biggest traps aren't about Mono/Flux mechanics — they're about
> assuming Spring 'just handles it' (404s, validation) when it actually needs
> you to be explicit."**
