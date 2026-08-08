# Streaming Between Microservices

## In Simple Terms

Beyond simple request/response calls, microservices can stream data to
each other continuously — one service producing a `Flux` that another
consumes in real time, using `WebClient` to read an NDJSON or SSE stream
from an upstream service.

## Simple Example

Producer service — streaming inventory updates:

```java
@GetMapping(value = "/inventory/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<InventoryUpdate> streamInventoryUpdates() {
    return inventoryEventBus.subscribe(); // Sinks-backed Flux of live inventory events
}
```

Consumer service — reading that stream reactively via `WebClient`:

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

This lets one service keep an always-up-to-date local view of another
service's state, continuously, without repeatedly polling for it.

## Why It Matters

Streaming between microservices (rather than polling with repeated
request/response calls) cuts latency for propagating changes and reduces
unnecessary network traffic — an increasingly common way to keep
distributed caches, search indexes, or read-replicas in sync with a
source-of-truth service close to real time.
