# Reactive Controllers

## In Simple Terms

A "reactive controller" is a regular Spring `@RestController`, but every method
returns a `Mono`/`Flux` instead of a plain object or blocking `List`. Aside from the
return types, the annotations (`@GetMapping`, `@PostMapping`, `@PathVariable`,
`@RequestBody`, etc.) all look and behave exactly like traditional Spring MVC.

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

Notice you can even accept `@RequestBody Mono<Order>` instead of a plain `Order` —
this lets Spring WebFlux start processing the request body reactively as it arrives
over the network, rather than waiting for the entire body to be fully read first.

## Why It Matters

Reactive controllers keep the same familiar Spring MVC annotation-based programming
model, minimizing the learning curve — the shift to WebFlux is mostly about *return
types* and *how you compose logic inside the method body* (using reactive operators
instead of blocking calls), not a wholesale rewrite of how controllers are declared.
