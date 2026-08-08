# Q1. Why Can't I Just Use JPA/Hibernate in a WebFlux App?

## Simple Explanation (Think of a One-Lane Bridge That Can't Be Widened)

JDBC — what JPA/Hibernate is built on — is a **one-lane bridge**: it was designed
decades ago so that a thread crosses it and simply **waits** until it's fully
across (the query finishes). No matter how cleverly you wrap it, you can't turn a
one-lane bridge into a multi-lane highway after the fact — the blocking nature is
baked into its foundation.

```
JDBC / JPA / Hibernate  =  fundamentally BLOCKING, no matter how you wrap it
R2DBC                    =  a completely different bridge, built non-blocking from day one
```

**R2DBC** (Reactive Relational Database Connectivity) is that new, wider bridge —
a separate specification and driver set, purpose-built for non-blocking database
access.

---

## Q2. Does Spring Data R2DBC Feel Different from Spring Data JPA?

**Barely.** If you know JPA, you already mostly know R2DBC:

```java
// JPA
public interface ProductRepository extends CrudRepository<ProductEntity, String> {
    List<ProductEntity> findByCategory(String category); // BLOCKING
}

// R2DBC — same method-name convention, just reactive return types
public interface ProductRepository extends ReactiveCrudRepository<ProductEntity, String> {
    Flux<ProductEntity> findByCategory(String category); // NON-BLOCKING
}
```

```java
Mono<ProductEntity> product = productRepository.findById("P123");
Flux<ProductEntity> allProducts = productRepository.findAll();
```

---

## Q3. What Do I Use for Complex, Dynamic SQL?

```java
// Simple: @Query annotation
@Query("SELECT * FROM products WHERE category = :category AND price < :maxPrice")
Flux<ProductEntity> findAffordableInCategory(String category, double maxPrice);

// Complex/dynamic: DatabaseClient — still fully reactive
public Flux<ProductEntity> searchProducts(String keyword, double minPrice) {
    return databaseClient.sql("SELECT * FROM products WHERE name ILIKE :keyword AND price >= :minPrice")
        .bind("keyword", "%" + keyword + "%")
        .bind("minPrice", minPrice)
        .map((row, meta) -> new ProductEntity(row.get("id", String.class), row.get("name", String.class), row.get("price", Double.class)))
        .all();
}
```

---

## Q4. What Actually Breaks If I Sneak JPA Into a WebFlux App?

```java
// This SILENTLY blocks an event-loop thread — even though the code "looks" reactive!
@GetMapping("/products/{id}")
public Mono<ProductDto> getProduct(@PathVariable String id) {
    return Mono.fromCallable(() -> jpaRepository.findById(id)) // JPA call is BLOCKING
        .map(ProductMapper::toDto);
    // Missing .subscribeOn(Schedulers.boundedElastic()) -> freezes an event-loop thread!
}
```

If you genuinely must use JPA (legacy system, no migration budget), you MUST
isolate it with `.subscribeOn(Schedulers.boundedElastic())` — otherwise it
silently undoes WebFlux's entire scalability benefit.

---

## Q5. Interview-Style Q&A

### Is R2DBC a wrapper around JDBC?

**No** — it's a completely separate specification/driver ecosystem, built
non-blocking from the ground up. JDBC cannot be made truly non-blocking, no matter
how it's wrapped.

### Can I mix R2DBC and JPA in the same application?

Technically yes, but any JPA call must be isolated on `boundedElastic()` — mixing
them without that isolation reintroduces blocking-thread problems.

### Does `ReactiveCrudRepository` support the same method-name query conventions as JPA's `CrudRepository`?

**Yes** — `findByX`, `countByX`, etc. all work the same way, just returning
`Mono`/`Flux` instead of plain values/collections.

---

## Q6. Summary

```
JDBC (blocking, can't be fixed)  →  JPA/Hibernate (blocking, built on JDBC)
                                              ✗ NOT suitable for pure WebFlux pipelines
                                                (unless isolated on boundedElastic())

R2DBC (genuinely non-blocking)  →  Spring Data R2DBC
                                              │
                          ReactiveCrudRepository (simple CRUD, method-name queries)
                                              │
                          DatabaseClient (complex/dynamic raw SQL, still reactive)
                                              │
                                     Mono / Flux results
```

### One sentence to remember

> **"JDBC is a one-lane bridge that can never be widened — if your WebFlux app
> talks to a relational database, it needs R2DBC, a completely different,
> genuinely non-blocking bridge."**
