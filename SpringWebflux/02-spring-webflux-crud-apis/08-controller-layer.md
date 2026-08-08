# Controller Layer

## In Simple Terms

The controller layer handles HTTP stuff only: mapping incoming requests to
service calls, and turning service results into HTTP responses (with the
right status codes). In a well-organized reactive app, controllers stay
thin — passing almost all the real logic off to the service layer.

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

Keeping HTTP-specific stuff (status codes, headers) isolated in the
controller — separate from business logic in the service layer — keeps
each layer focused and easy to test on its own: you can unit test
`ProductService` with no HTTP context at all, and test
`ProductController` with `WebTestClient`, focused purely on the HTTP side
of things.
