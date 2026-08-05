# R2DBC (Reactive Database) — Topic Overview

## What Is This Topic About? (In Simple Terms)

JDBC — the standard way Java talks to relational databases — is **fundamentally
blocking**, no matter how you wrap it. That means JPA/Hibernate, which sits on top
of JDBC, can never be truly non-blocking either. **R2DBC** (Reactive Relational
Database Connectivity) is a completely different specification and set of drivers,
built from the ground up around non-blocking I/O — the reactive counterpart to
JDBC.

The good news: if you already know Spring Data JPA, Spring Data R2DBC feels almost
identical — you extend `ReactiveCrudRepository` instead of `CrudRepository`, and
every method just returns `Mono`/`Flux` instead of a plain object or blocking
`List`:

```java
public interface ProductRepository extends ReactiveCrudRepository<ProductEntity, String> {
    Flux<ProductEntity> findByCategory(String category); // still works, just reactive
}

Mono<ProductEntity> product = productRepository.findById("P123");
Flux<ProductEntity> allProducts = productRepository.findAll();
```

For queries too complex for method-name conventions, `DatabaseClient` gives you a
reactive escape hatch for raw, dynamic SQL — still returning `Mono`/`Flux`, still
fully non-blocking.

**Why this matters so much:** using a blocking JPA repository inside an otherwise
reactive WebFlux pipeline would silently reintroduce the exact problem WebFlux was
meant to solve (a blocking call stalling a precious event-loop thread) — R2DBC keeps
your data access layer non-blocking end to end.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Reactive Repository** | `ReactiveCrudRepository<Entity, Id>` — Spring Data's reactive repository interface, Mono/Flux everywhere. |
| 2 | **Reactive CRUD** | `save()`, `findById()`, `findAll()`, `deleteById()`, `count()` — all built in, all reactive. |
| 3 | **R2DBC** | The reactive relational database spec/drivers — genuinely non-blocking, unlike JDBC. |
| 4 | **Mono/Flux database operations** | Every DB call returns Mono (0-1) or Flux (0-N) — composes naturally with the rest of your reactive code. |
| 5 | **Reactive SQL access** | `DatabaseClient` (or `@Query`) for complex/dynamic SQL beyond method-name conventions — still non-blocking. |

## How It All Fits Together

```
JDBC (blocking, can't be fixed)  →  JPA/Hibernate (blocking, built on JDBC)
                                              ✗ NOT suitable for pure WebFlux pipelines

R2DBC (genuinely non-blocking)  →  Spring Data R2DBC
                                              │
                          ReactiveCrudRepository (simple CRUD, method-name queries)
                                              │
                          DatabaseClient (complex/dynamic raw SQL, still reactive)
                                              │
                                     Mono / Flux results
                          composes directly with Service/Controller layers
```

Rule of thumb for this whole topic: **if your WebFlux app talks to a relational
database, it should be through R2DBC, not JDBC/JPA** — otherwise you're only
"half-reactive," with a blocking bottleneck hiding in your data layer.
