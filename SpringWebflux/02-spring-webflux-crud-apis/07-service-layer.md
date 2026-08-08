# Service Layer

## In Simple Terms

The service layer is where your business logic lives — validation,
coordinating multiple repository or external calls, converting entities to
DTOs — sitting between the controller (HTTP stuff) and the repository
(data access). In reactive apps, service methods also return
`Mono`/`Flux`, building on top of the repository's reactive results with
extra logic.

## Simple Example

```java
@Service
public class ProductService {

    private final ProductRepository repository;

    public ProductService(ProductRepository repository) {
        this.repository = repository;
    }

    public Mono<ProductDto> getProduct(String id) {
        return repository.findById(id)
            .switchIfEmpty(Mono.error(new ProductNotFoundException(id)))
            .map(ProductMapper::toDto);
    }

    public Mono<ProductDto> createProduct(ProductDto dto) {
        if (dto.price() <= 0) {
            return Mono.error(new IllegalArgumentException("Price must be positive"));
        }
        ProductEntity entity = ProductMapper.toEntity(dto);
        return repository.save(entity).map(ProductMapper::toDto);
    }
}
```

## Why It Matters

Keeping business logic in its own service layer (instead of stuffing it
into controllers) keeps controllers thin and focused purely on HTTP
concerns, while the service layer stays independently testable and
reusable — the same layering idea from traditional Spring apps, just
expressed reactively.
