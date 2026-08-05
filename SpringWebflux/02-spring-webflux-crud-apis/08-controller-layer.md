# Controller Layer

## In Simple Terms

The controller layer is responsible for **HTTP concerns only**: mapping incoming
requests to service calls, and translating service results into HTTP responses (with
appropriate status codes). In a well-layered reactive application, controllers stay
thin — delegating almost all real logic to the service layer.

## Simple Example

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    private final ProductService productService;

    public ProductController(ProductService productService) {
        this.productService = productService;
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<ProductDto>> getProduct(@PathVariable String id) {
        return productService.getProduct(id)
            .map(ResponseEntity::ok)
            .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    public Mono<ResponseEntity<ProductDto>> createProduct(@RequestBody ProductDto dto) {
        return productService.createProduct(dto)
            .map(created -> ResponseEntity.status(HttpStatus.CREATED).body(created));
    }
}
```

## Why It Matters

Keeping HTTP-specific logic (status codes, headers) isolated in the controller layer
— separate from business logic in the service layer — keeps each layer focused and
testable independently: you can unit test `ProductService` without any HTTP context,
and test `ProductController` with `WebTestClient` focused purely on the HTTP
contract.
