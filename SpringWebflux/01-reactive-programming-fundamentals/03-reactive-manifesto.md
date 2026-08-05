# Reactive Manifesto

## In Simple Terms

The **Reactive Manifesto** (reactivemanifesto.org) is a document describing the four
core traits a well-designed reactive system should have. Spring WebFlux and Project
Reactor are built with these principles in mind:

```
        Responsive
            ^
            |
Resilient <-+-> Elastic
            |
     Message Driven (the foundation)
```

- **Responsive**: the system responds in a timely manner, even under load.
- **Resilient**: the system stays responsive even when parts of it fail.
- **Elastic**: the system stays responsive under varying load, scaling up or down as
  needed.
- **Message Driven**: components communicate via asynchronous messages/events, which
  is what enables the other three traits.

## Simple Example

A reactive microservice architecture embodying these traits:

```java
public Mono<Dashboard> getDashboard(String userId) {
    return userService.getProfile(userId)
        .timeout(Duration.ofSeconds(1))                        // Responsive: bounded wait time
        .onErrorResume(e -> cachedProfileService.get(userId))  // Resilient: fallback on failure
        .zipWith(orderService.getRecentOrders(userId))          // Elastic: parallel, non-blocking calls
        .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2()));
        // Message Driven: everything here is async, event-based composition
}
```

## Why It Matters

The Reactive Manifesto isn't just marketing language — it's a practical checklist.
When evaluating whether a system is "truly reactive," ask: does it respond quickly
under load? Does it degrade gracefully on failure? Does it scale elastically? Is its
internal communication asynchronous and message-driven? Spring WebFlux gives you the
tools to build systems that check all four boxes.
