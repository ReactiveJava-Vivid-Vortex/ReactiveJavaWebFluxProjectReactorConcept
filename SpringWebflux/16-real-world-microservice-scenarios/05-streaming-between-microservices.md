# Streaming Between Microservices

## In Simple Terms

Beyond simple request/response calls, microservices can stream data to each other
continuously — one service producing a `Flux` that another consumes in real time,
using WebClient to consume an NDJSON or SSE stream from an upstream service.

## Simple Example

Producer service — streaming inventory updates:

```java
@GetMapping(value = "/inventory/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<InventoryUpdate> streamInventoryUpdates() {
    return inventoryEventBus.subscribe(); // Sinks-backed Flux of live inventory events
}
```

Consumer service — consuming that stream reactively via WebClient:

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

This lets one service maintain an up-to-date local view of another service's state,
continuously, without needing to poll repeatedly.

## Why It Matters

Streaming between microservices (rather than repeated request/response polling)
reduces latency for propagating changes and cuts down on unnecessary network
traffic — an increasingly common pattern for keeping distributed caches, search
indexes, or read-replicas synchronized with a source-of-truth service in near
real time.
