# Reactive Controllers

## In Simple Terms

A "reactive controller" is just a regular Spring `@RestController` where
every method returns a `Mono`/`Flux` instead of a plain object or a
blocking `List`. Aside from the return types, all the familiar annotations
(`@GetMapping`, `@PostMapping`, `@PathVariable`, `@RequestBody`) look and
work exactly like traditional Spring MVC.

## Simple Example

```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping("/{id}")
    public Mono<Order> getOrder(@PathVariable String id) {
        return orderService.findById(id);
    }

    @PostMapping
    public Mono<Order> createOrder(@RequestBody Mono<Order> orderMono) {
        return orderMono.flatMap(orderService::create);
    }
}
```

You can even accept `@RequestBody Mono<Order>` instead of a plain `Order`
— this lets WebFlux start reactively processing the request body as it
arrives over the network, rather than waiting for the whole body to load
first.

## Why It Matters

Reactive controllers keep the same familiar Spring MVC annotation style,
which keeps the learning curve small — moving to WebFlux is mostly about
*return types* and *how you write the logic inside* (reactive operators
instead of blocking calls), not a whole new way of declaring controllers.
