# Functional and Annotated Endpoints Can Coexist

## In Simple Terms

A common misconception: "you have to pick one style — either `@RestController`
annotations, or `RouterFunction`/functional endpoints — for your whole
application." **Not true.** Spring WebFlux happily runs both styles side by side,
in the same application, at the same time. Nothing stops you from having some
resources use annotated controllers and others use functional routing.

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

Both sets of endpoints are registered and served by the same WebFlux application,
using the same underlying `HandlerMapping` infrastructure — there's no conflict or
special configuration needed to mix them.

## Why It Matters

This matters most during **migrations** — you can incrementally convert an
annotation-based API to functional routing (or vice versa) one resource at a time,
without a disruptive big-bang rewrite. It also means teams can pick whichever
style best fits a specific endpoint's complexity (simple CRUD often reads cleaner
with annotations; endpoints needing fine-grained control over routing/validation
logic sometimes read cleaner as explicit functions) without being locked into one
approach for the entire codebase.
