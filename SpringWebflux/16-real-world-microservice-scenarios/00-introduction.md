# Q1. What Happens When a Service You Depend On Actually Fails?

## Simple Explanation (Think of a Relay Race Where One Runner Might Trip)

In a microservices architecture, assuming every downstream call succeeds is like
assuming no runner in a relay race will ever trip. It's not a hypothetical — it's
a certainty over enough races. This topic is about **designing for the trip**,
not hoping it never happens.

```java
public Mono<InventoryStatus> checkInventory(String productId) {
    return inventoryServiceClient.checkStock(productId)
        .timeout(Duration.ofSeconds(1))                        // bound the wait
        .retryWhen(Retry.backoff(2, Duration.ofMillis(100)))    // retry transient blips
        .onErrorResume(error -> {                                // final fallback
            log.warn("Inventory check failed, assuming available", error);
            return Mono.just(InventoryStatus.assumeAvailable());
        });
}
```

---

## Q2. What Is a "Partial Response," and Why Is It Better Than Failing Everything?

```java
public record Dashboard(Optional<UserProfile> profile, Optional<List<Order>> orders) {}

public Mono<Dashboard> getDashboard(String userId) {
    Mono<Optional<UserProfile>> profileMono = userService.getProfile(userId)
        .map(Optional::of).onErrorReturn(Optional.empty());
    Mono<Optional<List<Order>>> ordersMono = orderService.getRecentOrders(userId)
        .map(Optional::of).onErrorReturn(Optional.empty());

    return Mono.zip(profileMono, ordersMono)
        .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2()));
}
```

If the orders service is down, the dashboard still renders with the profile —
**partial data beats a blank error page.**

---

## Q3. What's the Difference Between "Resilience" and "Graceful Degradation"?

| Concept | Question It Answers | Example |
|---|---|---|
| Resilience | "How do I survive the failure?" | timeout + retry + circuit breaker + fallback |
| Graceful degradation | "What should the user see while degraded?" | Basic search instead of ML-powered search |

```java
public Mono<SearchResults> search(String query) {
    return advancedSearchService.search(query)
        .timeout(Duration.ofSeconds(1))
        .onErrorResume(error -> basicSearchService.search(query)); // simpler, but reliable
}
```

---

## Q4. What Does a "Defense in Depth" Resilience Stack Look Like?

```java
public Mono<InventoryStatus> checkInventory(String productId) {
    return inventoryServiceClient.checkStock(productId)
        .timeout(Duration.ofSeconds(1))                          // Responsive
        .retryWhen(Retry.backoff(2, Duration.ofMillis(100))      // Resilient (transient blips)
            .filter(e -> e instanceof TimeoutException))
        .transform(CircuitBreakerOperator.of(circuitBreaker))     // Resilient (sustained outages)
        .onErrorResume(error -> Mono.just(InventoryStatus.assumeAvailable())); // last resort
}
```

Each layer handles a **different** failure mode — timeout handles slowness, retry
handles blips, circuit breaker handles sustained outages, fallback handles
everything else.

---

## Q5. How Do Microservices Stay Synchronized Without Constant Polling?

```java
public Flux<InventoryUpdate> subscribeToInventoryUpdates() {
    return webClient.get()
        .uri("http://inventory-service/inventory/stream")
        .accept(MediaType.APPLICATION_NDJSON)
        .retrieve()
        .bodyToFlux(InventoryUpdate.class)
        .doOnNext(update -> localCache.applyUpdate(update))
        .retryWhen(Retry.backoff(Long.MAX_VALUE, Duration.ofSeconds(1))); // reconnect indefinitely
}
```

Streaming (NDJSON/SSE) + indefinite reconnection keeps a local cache synchronized
with an upstream source of truth in near real time.

---

## Q6. Interview-Style Q&A

### Should every downstream call have a timeout?

**Yes, always** — without exception. An unbounded wait on any external call is a
latent production incident.

### What's the risk of `retryWhen()` without backoff?

Hammering an already-struggling service with immediate retries can make the
outage worse — always use exponential backoff for real external dependencies.

### If a service aggregates 3 downstream calls and 1 fails, should the whole request fail?

**Usually no** — wrap each call's failure in `Optional`/fallback and return a
partial response, unless that specific piece of data is truly mandatory.

---

## Q7. Summary

```
Call to downstream service
        │
        ▼
.timeout(...)              ← bound the wait (Responsive)
        │
        ▼
.retryWhen(backoff)         ← survive transient blips (Resilient)
        │
        ▼
.onErrorResume(fallback)    ← degrade gracefully instead of failing outright
        │
        ▼
Aggregating multiple such calls? → wrap each in Optional, return a PARTIAL response
        │
        ▼
Need continuous sync instead of one-off calls? → stream via WebClient + NDJSON/SSE
```

### One sentence to remember

> **"Something will eventually fail in a distributed system — the only
> question is whether you designed for it (timeout → retry → fallback) or
> found out in production."**
