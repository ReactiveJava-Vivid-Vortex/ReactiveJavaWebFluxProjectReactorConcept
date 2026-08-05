# Real-World Microservice Scenarios — Topic Overview

## What Is This Topic About? (In Simple Terms)

This final topic is where every earlier concept — timeouts, retries, fallbacks,
WebClient, error handling — comes together to answer one practical question: **what
should actually happen when a downstream service you depend on fails?** In a
microservices architecture, this isn't a hypothetical — it's a certainty. Networks
drop, services crash, dependencies time out.

A resilient service layers several defenses together rather than relying on just
one:

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

Beyond simple fallbacks, well-designed services aggregate data from multiple
sources and return **partial responses** — if the recommendations service is down
but the profile and orders services are up, a dashboard still renders with what's
available rather than failing completely. This is **graceful degradation**:
maintaining reduced but still-useful functionality instead of an all-or-nothing
outcome.

Finally, microservices increasingly **stream** data to each other continuously
(via NDJSON/SSE, consumed with `WebClient` + `.retryWhen()` for indefinite
reconnection) rather than repeatedly polling — keeping a local cache or view
synchronized with an upstream source of truth in near real time.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Downstream service failures** | Always pair a timeout + retry + fallback around any call to another service — never assume it will succeed. |
| 2 | **Partial responses** | When aggregating multiple sources, return what succeeded (wrapped in `Optional`) instead of failing everything. |
| 3 | **Graceful degradation** | Fall back to a simpler, more reliable alternative (cache, basic search) rather than breaking the feature entirely. |
| 4 | **Resilience** | Layer timeout + retry-with-backoff + circuit breaker + fallback together for defense in depth. |
| 5 | **Streaming between microservices** | Use NDJSON/SSE + WebClient (with indefinite `.retryWhen()`) to keep services synchronized continuously, instead of polling. |

## How It All Fits Together

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

This topic is the practical payoff of the entire course: every pattern here is just
Project Reactor's error-handling and WebClient operators, applied deliberately to
the one guarantee of distributed systems — **something will eventually fail, so
design for it up front.**
