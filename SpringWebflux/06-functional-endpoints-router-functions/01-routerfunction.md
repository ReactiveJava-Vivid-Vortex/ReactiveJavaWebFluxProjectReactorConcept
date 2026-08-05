# RouterFunction

## In Simple Terms

A `RouterFunction<ServerResponse>` is the functional-style alternative to
`@RestController`/`@GetMapping` — instead of annotations, you explicitly declare
routes (URL patterns + HTTP methods) mapped to handler functions, using a fluent
builder API. Many production WebFlux projects prefer this style for its explicitness
and testability.

## Simple Example

```java
@Configuration
public class ProductRoutes {

    @Bean
    public RouterFunction<ServerResponse> productRoutes(ProductHandler handler) {
        return RouterFunctions.route()
            .GET("/products", handler::getAllProducts)
            .GET("/products/{id}", handler::getProduct)
            .POST("/products", handler::createProduct)
            .PUT("/products/{id}", handler::updateProduct)
            .DELETE("/products/{id}", handler::deleteProduct)
            .build();
    }
}
```

Compare with the annotation-based equivalent (functionally identical, different
style):

```java
@RestController
@RequestMapping("/products")
public class ProductController {
    @GetMapping public Flux<Product> getAllProducts() { ... }
    @GetMapping("/{id}") public Mono<Product> getProduct(...) { ... }
    // ...
}
```

## Why It Matters

`RouterFunction` makes routing configuration explicit and centralized — all your
routes are visible in one place, rather than scattered across annotations on
individual controller methods — which many teams find easier to review, test, and
reason about as an API surface grows large.
