# Spring WebFlux CRUD APIs — Topic Overview

## What Is This Topic About? (In Simple Terms)

This topic is where the theory becomes real code: building a standard CRUD
(Create/Read/Update/Delete) REST API using Spring WebFlux. The good news — if you
already know Spring MVC, the annotations are **identical**
(`@RestController`, `@GetMapping`, `@PathVariable`, `@RequestBody`). The only real
change is the return type: every method returns `Mono<T>` (0-1 result) or
`Flux<T>` (0-N results) instead of a plain object or blocking `List`.

```java
@RestController
@RequestMapping("/products")
public class ProductController {

    @GetMapping("/{id}")
    public Mono<ProductDto> getProduct(@PathVariable String id) {
        return productService.getProduct(id); // 0 or 1 result
    }

    @GetMapping
    public Flux<ProductDto> getAllProducts() {
        return productService.getAllProducts(); // 0 to N results
    }
}
```

The other core habit this topic builds is proper **layering**: Controller (HTTP
concerns only — status codes, headers) → Service (business logic, validation,
entity-to-DTO mapping) → Repository (reactive data access, returning `Mono`/`Flux`).
Keeping these separate — rather than putting everything in the controller — keeps
each layer independently testable.

A recurring, important detail across every endpoint: correctly handling the "empty"
case. `findById()` returning empty should become a clear `404 Not Found`, not a
silent, ambiguous response:

```java
return productRepository.findById(id)
    .map(ResponseEntity::ok)
    .defaultIfEmpty(ResponseEntity.notFound().build());
```

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Mono** | Controller return type for 0-1 results — empty Mono typically means 404. |
| 2 | **Flux** | Controller return type for 0-N results — can be a JSON array or streamed (NDJSON/SSE). |
| 3 | **Reactive Controllers** | Same annotations as Spring MVC (`@GetMapping`, etc.) — just Mono/Flux return types. |
| 4 | **DTOs** | Plain objects shaped for the API, decoupled from your internal database Entity. |
| 5 | **Entity Mapping** | Converting Entity ↔ DTO in both directions, usually via `.map()` inside the reactive chain. |
| 6 | **Repository Layer** | `ReactiveCrudRepository` — reactive data access, every method returns Mono/Flux. |
| 7 | **Service Layer** | Business logic, validation, orchestration — sits between controller and repository. |
| 8 | **Controller Layer** | HTTP concerns only (status codes, headers) — stays thin, delegates to the service layer. |
| 9 | **GET All** | Return `Flux<Dto>` from `repository.findAll()`, optionally filtered/paginated. |
| 10 | **GET By Id** | Return `Mono<Dto>`; explicitly handle the empty case as 404 via `defaultIfEmpty()`. |
| 11 | **POST** | Save via repository, return `201 Created` with the saved (server-populated) entity. |
| 12 | **PUT** | Look up existing (via `flatMap`), apply updates, save; 404 if the id doesn't exist. |
| 13 | **DELETE** | `repository.deleteById()` returns `Mono<Void>`; typically respond with `204 No Content`. |

## How It All Fits Together

```
HTTP Request
     │
     ▼
Controller Layer   (HTTP concerns: status codes, @PathVariable/@RequestBody)
     │
     ▼
Service Layer      (business logic, validation, Entity ↔ DTO mapping)
     │
     ▼
Repository Layer   (ReactiveCrudRepository — Mono/Flux database access)
     │
     ▼
Response streamed back (Mono/Flux, non-blocking end to end)
```

Once these five CRUD operations feel natural in Mono/Flux terms, every other
WebFlux topic (validation, error handling, filters) is just adding more
sophistication on top of this same basic layered shape.
