# Functional Routing

## In Simple Terms

"Functional routing" is the overall style of declaring your API's routes using
`RouterFunction` composition, rather than scattering `@GetMapping`/`@PostMapping`
annotations across controller classes. Routes are composed together using methods
like `.and()`, `.andRoute()`, or nested `RouterFunctions.route()` calls.

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

Functional routing's composability makes it easy to organize large APIs into
modular, independently-defined route groups (one per resource/feature), and combine
them together explicitly — giving you a single, traceable source of truth for your
entire application's routing table.
