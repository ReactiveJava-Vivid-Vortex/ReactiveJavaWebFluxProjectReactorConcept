# Non-Blocking Execution (WebFlux)

## In Simple Terms

As covered in depth in the Project Reactor notes
([[non-blocking-execution]]), the single most important rule for keeping
WebFlux's performance benefits is making sure every part of your
request-handling pipeline — controllers, services, repositories, external
calls — stays genuinely non-blocking, end to end.

## Simple Example

A full non-blocking pipeline, from controller to database to external
service:

```java
@GetMapping("/orders/{id}/full-details")
public Mono<OrderDetails> getOrderDetails(@PathVariable String id) {
    return orderRepository.findById(id)                 // non-blocking: R2DBC
        .flatMap(order -> customerServiceClient          // non-blocking: WebClient
            .getCustomer(order.getCustomerId())
            .map(customer -> new OrderDetails(order, customer)));
}
```

Every single step here — the database lookup and the downstream service
call — is non-blocking, meaning no thread ever sits frozen waiting
anywhere in this chain.

## Why It Matters

A single blocking call anywhere in an otherwise reactive pipeline (a
leftover blocking JDBC call, a synchronous file read, `Thread.sleep()`)
can quietly drag down performance for the whole app, since it stalls one
of the small handful of event-loop threads shared across many concurrent
requests. Regularly checking your whole call chain for hidden blocking
calls is worth doing continuously, not just once.
