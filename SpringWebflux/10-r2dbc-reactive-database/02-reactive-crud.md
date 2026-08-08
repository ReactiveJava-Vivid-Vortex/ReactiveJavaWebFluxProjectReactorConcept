# Reactive CRUD

## In Simple Terms

`ReactiveCrudRepository` gives you all the standard CRUD operations
(Create/Read/Update/Delete) right out of the box — `save()`, `findById()`,
`findAll()`, `deleteById()`, `count()`, and more — each returning
`Mono`/`Flux`, following the same Spring Data model you already know from
JPA.

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

Getting all the standard CRUD operations for free — just by extending one
interface — means you can spin up a fully working reactive data layer for
a new entity in minutes, exactly like traditional Spring Data JPA, just
non-blocking from the ground up.
