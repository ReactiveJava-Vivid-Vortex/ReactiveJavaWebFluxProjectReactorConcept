# Cancellation of Requests

## In Simple Terms

One of WebFlux's underrated efficiency wins: if a client disconnects mid-request
(closes the browser tab, network drops, or the HTTP connection is otherwise
terminated), Spring WebFlux **automatically cancels** the underlying reactive
pipeline — stopping any further processing that would just be wasted work for a
client that's no longer listening.

## Simple Example

```java
@GetMapping(value = "/live-feed", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> liveFeed() {
    return Flux.interval(Duration.ofSeconds(1))
        .doOnCancel(() -> System.out.println("Client disconnected - stopping feed"))
        .map(tick -> "Update #" + tick);
}
```

If a user opens this endpoint in their browser and then closes the tab, "Client
disconnected - stopping feed" prints automatically — WebFlux propagates the
connection close as a `cancel()` signal down through the entire reactive pipeline,
including stopping the `Flux.interval()` ticker.

## Why It Matters

In a traditional blocking server, a similar scenario (a slow client, or one that
disconnects early) might still leave the server-side thread and any in-progress work
running until it naturally finishes — wasting resources on work nobody will ever
receive. WebFlux's automatic cancellation propagation means abandoned work is cleaned
up promptly, which matters a lot for infinite or long-running streams (like SSE feeds
or large file downloads).
