# Non-Blocking Execution (WebFlux)

## In Simple Terms

As covered in depth in the ProjectReactor notes ([[non-blocking-execution]]), the
single most important rule for maintaining WebFlux's performance benefits is
ensuring every part of your request-handling pipeline — controllers, services,
repositories, external calls — remains genuinely non-blocking, end to end.

## Simple Example

A full non-blocking pipeline, from controller to database to external service:

```java
@GetMapping("/orders/{id}/full-details")
public Mono<OrderDetails> getOrderDetails(@PathVariable String id) {
    return orderRepository.findById(id)                 // non-blocking: R2DBC
        .flatMap(order -> customerServiceClient          // non-blocking: WebClient
            .getCustomer(order.getCustomerId())
            .map(customer -> new OrderDetails(order, customer)));
}
```

Every single step here — the database lookup and the downstream service call — is
non-blocking, meaning no thread is ever frozen waiting at any point in this chain.

## Why It Matters

A single blocking call anywhere in an otherwise-reactive pipeline (a leftover
blocking JDBC call, a synchronous file read, `Thread.sleep()`) can silently degrade
performance for the entire application, since it stalls one of the small number of
event-loop threads shared across many concurrent requests. Auditing your entire
call chain for hidden blocking calls is an essential, ongoing practice.
