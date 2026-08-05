# Reactive Pipeline

## In Simple Terms

In the context of Spring WebFlux, a "reactive pipeline" refers to the entire chain of
processing a single HTTP request goes through — from the incoming request, through
your controller's `Mono`/`Flux`-returning method, through any repository/WebClient
calls, all the way to the outgoing HTTP response — all connected as one continuous,
non-blocking, asynchronous chain.

## Simple Example

```java
@RestController
public class OrderController {

    @GetMapping("/orders/{id}/summary")
    public Mono<OrderSummary> getOrderSummary(@PathVariable String id) {
        return orderRepository.findById(id)                 // reactive DB call
            .flatMap(order -> userService.getUser(order.getUserId())  // reactive HTTP call
                .map(user -> new OrderSummary(order, user)))
            .switchIfEmpty(Mono.error(new OrderNotFoundException(id)))
            .doOnNext(summary -> log.info("Built summary for order {}", id));
    }
}
```

This entire method describes one continuous reactive pipeline: WebFlux subscribes to
it when the HTTP request arrives, data flows through the database call, the
downstream user service call, the combination logic, and finally to the HTTP
response — with no thread ever blocked at any stage.

## Why It Matters

Thinking of your entire request-handling logic as "one reactive pipeline" (rather
than a sequence of separate blocking steps) is the core mental shift needed to write
idiomatic WebFlux code — every operator you chain (`flatMap`, `switchIfEmpty`,
`doOnNext`) is a stage in that same, single, non-blocking pipeline.
