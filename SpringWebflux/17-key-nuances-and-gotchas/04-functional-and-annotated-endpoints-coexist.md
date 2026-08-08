# Functional and Annotated Endpoints Can Coexist

## In Simple Terms

A common misconception: "you have to pick one style — either
`@RestController` annotations, or `RouterFunction`/functional endpoints —
for the whole app." Not true. Spring WebFlux happily runs both styles side
by side, in the same application, at the same time. Nothing stops you from
having some resources use annotated controllers and others use functional
routing.

## Simple Example

```java
// Annotated style, for one resource
@RestController
@RequestMapping("/products")
public class ProductController {
    @GetMapping("/{id}")
    public Mono<ProductDto> getProduct(@PathVariable String id) { ... }
}
```

```java
// Functional style, for a different resource — same application, no conflict
@Configuration
public class OrderRoutes {
    @Bean
    public RouterFunction<ServerResponse> orderRoutes(OrderHandler handler) {
        return RouterFunctions.route()
            .GET("/orders/{id}", handler::getOrder)
            .build();
    }
}
```

Both sets of endpoints get registered and served by the same WebFlux
application, using the same underlying `HandlerMapping` machinery — no
special configuration needed to mix them.

## Why It Matters

This is especially useful during migrations — you can gradually move an
annotation-based API over to functional routing (or the other way around)
one resource at a time, instead of a disruptive all-at-once rewrite. It
also means teams can pick whichever style fits a given endpoint's
complexity best (simple CRUD often reads cleaner with annotations;
endpoints needing precise control over routing or validation sometimes
read cleaner as explicit functions) without being locked into one approach
for the whole codebase.
