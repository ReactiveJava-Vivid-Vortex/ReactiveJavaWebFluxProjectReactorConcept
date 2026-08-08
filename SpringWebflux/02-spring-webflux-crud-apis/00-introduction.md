# Q1. What Changes When You Build CRUD APIs in WebFlux vs Spring MVC?

## Simple Explanation (Think of Swapping a Delivery Van's Engine, Not Its Body)

Good news first: if you already know Spring MVC, WebFlux controllers look almost
identical. Same annotations, same layered architecture. The only real change is
the "engine" — return types become `Mono`/`Flux` instead of plain objects.

```java
// Spring MVC "body"                       // WebFlux "body" — SAME shape
@RestController                             @RestController
@RequestMapping("/products")                 @RequestMapping("/products")
public class ProductController {             public class ProductController {

  @GetMapping("/{id}")                          @GetMapping("/{id}")
  public ProductDto getProduct(...) { }          public Mono<ProductDto> getProduct(...) { }
                                                                  ↑ "engine swap"
```

---

## Q2. What's the Standard Layering, and Who's Responsible for What?

```
HTTP Request
     │
     ▼
Controller Layer   — HTTP concerns ONLY (status codes, @PathVariable/@RequestBody)
     │
     ▼
Service Layer      — business logic, validation, Entity ↔ DTO mapping
     │
     ▼
Repository Layer   — ReactiveCrudRepository, Mono/Flux database access
```

```java
@RestController @RequestMapping("/products")
public class ProductController {                       // Controller: thin, HTTP only
    @GetMapping("/{id}")
    public Mono<ProductDto> getProduct(@PathVariable String id) {
        return productService.getProduct(id);
    }
}

@Service
public class ProductService {                          // Service: business logic
    public Mono<ProductDto> getProduct(String id) {
        return repository.findById(id)
            .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
            .map(ProductMapper::toDto);
    }
}
```

---

## Q3. Mono or Flux — How Do I Choose the Return Type?

| Endpoint Returns... | Type |
|---|---|
| One item, or nothing (GET by id, POST result) | `Mono<T>` |
| Zero-to-many items (GET all, search results) | `Flux<T>` |
| No meaningful result (DELETE) | `Mono<Void>` |

---

## Q4. GET By Id — The Empty-Case Trap

```java
@GetMapping("/{id}")
public Mono<ResponseEntity<ProductDto>> getProduct(@PathVariable String id) {
    return productRepository.findById(id)
        .map(ProductMapper::toDto)
        .map(ResponseEntity::ok)
        .defaultIfEmpty(ResponseEntity.notFound().build()); // MUST be explicit — see Q8!
}
```

---

## Q5. POST — Creating a Resource

```java
@PostMapping
public Mono<ResponseEntity<ProductDto>> createProduct(@RequestBody ProductDto dto) {
    return productRepository.save(ProductMapper.toEntity(dto))
        .map(ProductMapper::toDto)
        .map(saved -> ResponseEntity.status(HttpStatus.CREATED).body(saved));
}
```

---

## Q6. PUT — Updating, With the "Does It Exist" Check

```java
@PutMapping("/{id}")
public Mono<ResponseEntity<ProductDto>> updateProduct(@PathVariable String id, @RequestBody ProductDto dto) {
    return productRepository.findById(id)
        .flatMap(existing -> productRepository.save(new ProductEntity(id, dto.name(), dto.price())))
        .map(ProductMapper::toDto)
        .map(ResponseEntity::ok)
        .defaultIfEmpty(ResponseEntity.notFound().build()); // 404 if id doesn't exist
}
```

Notice `.flatMap()` — looking up the existing entity is itself async, so `.map()`
would give you a `Mono<Mono<Entity>>` instead of a flattened `Mono<Entity>`.

---

## Q7. DELETE — What Does It Actually Return?

```java
@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public Mono<Void> deleteProduct(@PathVariable String id) {
    return productRepository.deleteById(id); // Mono<Void> — no meaningful value, just completion
}
```

---

## Q8. Interview-Style Q&A

### If a repository's `findById()` completes empty, does WebFlux automatically return 404?

**No** — a huge, common trap. By default it returns `200 OK` with an empty body.
You must explicitly convert the empty case via `.defaultIfEmpty(ResponseEntity.notFound()...)`.

### What's the difference between `.map()` and `.flatMap()` in a service method?

`.map()` is for **synchronous** transformations. `.flatMap()` is for chaining
**another async operation** (another `Mono`/`Flux`-returning call) — using `.map()`
there would nest Monos incorrectly.

### Should business logic live in the controller?

**No** — keep controllers thin (HTTP concerns only); put business logic, DTO
mapping, and validation in the service layer for testability.

---

## Q9. Summary

| Endpoint | Return Type | Key Detail |
|---|---|---|
| GET all | `Flux<Dto>` | Optionally filtered/paginated |
| GET by id | `Mono<Dto>` (or `Mono<ResponseEntity<Dto>>`) | Explicitly handle empty → 404 |
| POST | `Mono<Dto>` | Respond `201 Created` |
| PUT | `Mono<Dto>` | `flatMap()` the existence check; 404 if missing |
| DELETE | `Mono<Void>` | Respond `204 No Content` |

### One sentence to remember

> **"Same layered architecture as Spring MVC — the only real change is the
> engine: Mono/Flux instead of plain objects, and you must handle 'empty'
> explicitly instead of assuming a default."**
