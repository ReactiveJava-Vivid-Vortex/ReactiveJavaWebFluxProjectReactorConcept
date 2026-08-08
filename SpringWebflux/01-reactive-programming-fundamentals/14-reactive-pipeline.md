# Reactive Pipeline

## In Simple Terms

In Spring WebFlux, a "reactive pipeline" is the entire journey a single
HTTP request takes — from the incoming request, through your controller's
`Mono`/`Flux`-returning method, through any repository or `WebClient`
calls, all the way to the outgoing response — all wired together as one
continuous, non-blocking chain.

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

This whole method describes one continuous pipeline: WebFlux subscribes to
it the moment the HTTP request arrives, and data flows through the
database call, the downstream user-service call, the combining logic, and
finally out to the HTTP response — with no thread ever frozen at any point
along the way.

## Why It Matters

Thinking of your entire request-handling logic as "one pipeline" — rather
than a series of separate blocking steps — is the core shift needed to
write natural WebFlux code. Every operator you chain (`flatMap`,
`switchIfEmpty`, `doOnNext`) is just one stage in that same, single,
non-blocking pipeline.
