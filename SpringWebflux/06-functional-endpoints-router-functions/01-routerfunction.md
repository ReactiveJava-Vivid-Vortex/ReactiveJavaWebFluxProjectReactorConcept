# RouterFunction

## In Simple Terms

A `RouterFunction<ServerResponse>` is the functional-style alternative to
`@RestController`/`@GetMapping` — instead of annotations, you write out
your routes (URL patterns plus HTTP methods) explicitly, mapped to handler
functions, using a fluent builder. Many production WebFlux projects prefer
this style because it's explicit and easy to test.

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

Compare with the annotation-based version (does the exact same thing,
different style):

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

`RouterFunction` makes your routing explicit and all in one place — every
route is visible together, instead of scattered across annotations on
different controller methods — which many teams find easier to review,
test, and reason about as an API grows.
