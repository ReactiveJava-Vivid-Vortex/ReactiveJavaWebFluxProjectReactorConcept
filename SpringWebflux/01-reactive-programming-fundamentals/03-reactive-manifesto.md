# Reactive Manifesto

## In Simple Terms

The Reactive Manifesto (reactivemanifesto.org) lays out four traits a
well-built reactive system should have. Spring WebFlux and Project Reactor
are both designed with these in mind:

```
        Responsive
            ^
            |
Resilient <-+-> Elastic
            |
     Message Driven (the foundation)
```

- **Responsive**: answers in a timely way, even when things get busy.
- **Resilient**: keeps working even when some part of it breaks.
- **Elastic**: handles more or less load by scaling up or down as needed.
- **Message Driven**: parts of the system talk to each other through
  async messages/events, which is what makes the other three possible.

## Simple Example

A reactive microservice putting all four traits into practice:

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

The Reactive Manifesto isn't just buzzwords — it's a practical checklist.
When you're wondering if a system is "truly reactive," ask: does it
respond quickly under load? Does it degrade gracefully when something
fails? Does it scale up and down smoothly? Does it talk internally through
async messages? Spring WebFlux gives you what you need to check all four
boxes.
