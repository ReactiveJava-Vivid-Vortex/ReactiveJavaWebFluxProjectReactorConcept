# Reactive CRUD

## In Simple Terms

`ReactiveCrudRepository` provides all the standard CRUD operations
(Create/Read/Update/Delete) out of the box — `save()`, `findById()`, `findAll()`,
`deleteById()`, `count()`, and more — each returning `Mono`/`Flux`, matching the
familiar Spring Data programming model you'd already know from JPA.

## Simple Example

```java
public interface ProductRepository extends ReactiveCrudRepository<ProductEntity, String> {
}

@Service
public class ProductService {
    private final ProductRepository repository;

    public Mono<ProductEntity> create(ProductEntity product) {
        return repository.save(product); // INSERT (or UPDATE if id already exists)
    }

    public Mono<ProductEntity> getById(String id) {
        return repository.findById(id);
    }

    public Flux<ProductEntity> getAll() {
        return repository.findAll();
    }

    public Mono<Void> delete(String id) {
        return repository.deleteById(id);
    }
}
```

## Why It Matters

Getting all standard CRUD operations "for free" — just by extending an interface —
means you can build a fully-functional reactive data access layer for a new entity
in minutes, exactly like with traditional Spring Data JPA, just non-blocking from
the ground up.
