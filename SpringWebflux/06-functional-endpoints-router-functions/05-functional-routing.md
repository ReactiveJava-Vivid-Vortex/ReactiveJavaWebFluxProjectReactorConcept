# Functional Routing

## In Simple Terms

"Functional routing" is the overall style of declaring your API's routes
by composing `RouterFunction`s, instead of scattering
`@GetMapping`/`@PostMapping` annotations across controller classes. You
compose routes together with methods like `.and()`, `.andRoute()`, or by
nesting `RouterFunctions.route()` calls.

## Simple Example

```java
@Bean
public RouterFunction<ServerResponse> allRoutes(
        ProductHandler productHandler,
        OrderHandler orderHandler) {

    RouterFunction<ServerResponse> productRoutes = RouterFunctions.route()
        .GET("/products", productHandler::getAll)
        .GET("/products/{id}", productHandler::getById)
        .build();

    RouterFunction<ServerResponse> orderRoutes = RouterFunctions.route()
        .GET("/orders", orderHandler::getAll)
        .POST("/orders", orderHandler::create)
        .build();

    return productRoutes.and(orderRoutes); // compose multiple RouterFunctions together
}
```

You can also nest routes under a shared path prefix:

```java
RouterFunctions.route()
    .path("/products", builder -> builder
        .GET(productHandler::getAll)
        .GET("/{id}", productHandler::getById)
    )
    .build();
```

## Why It Matters

Because functional routes compose so easily, it's simple to organize a
large API into separate, independently-defined route groups (one per
resource or feature) and combine them explicitly — giving you a single,
traceable source of truth for how your whole app's routing works.
