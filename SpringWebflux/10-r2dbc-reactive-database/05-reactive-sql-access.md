# Reactive SQL Access

## In Simple Terms

For queries too complex for `ReactiveCrudRepository`'s method-name
conventions or simple `@Query` annotations, Spring Data R2DBC gives you
`R2dbcEntityTemplate` (or the lower-level `DatabaseClient`) to write raw,
custom SQL reactively — still returning `Mono`/`Flux`, still fully
non-blocking.

## Simple Example

Using `@Query` for a simple custom query:

```java
public interface ProductRepository extends ReactiveCrudRepository<ProductEntity, String> {

    @Query("SELECT * FROM products WHERE category = :category AND price < :maxPrice")
    Flux<ProductEntity> findAffordableInCategory(String category, double maxPrice);
}
```

Using `DatabaseClient` directly for more dynamic, complex SQL:

```java
@Repository
public class ProductQueryRepository {

    private final DatabaseClient databaseClient;

    public Flux<ProductEntity> searchProducts(String keyword, double minPrice) {
        return databaseClient.sql(
                "SELECT * FROM products WHERE name ILIKE :keyword AND price >= :minPrice"
            )
            .bind("keyword", "%" + keyword + "%")
            .bind("minPrice", minPrice)
            .map((row, metadata) -> new ProductEntity(
                row.get("id", String.class),
                row.get("name", String.class),
                row.get("price", Double.class)
            ))
            .all(); // returns a Flux<ProductEntity>
    }
}
```

## Why It Matters

Having an escape hatch (`DatabaseClient`) for complex, dynamic SQL — while
staying fully reactive the whole time — means you never need to fall back
to a blocking JDBC call just because a query is too complicated for
repository method-name conventions or a simple `@Query`.
