# Cancellation of Requests

## In Simple Terms

One of WebFlux's under-appreciated perks: if a client disconnects mid-way
through a request (closes the browser tab, network drops, connection gets
cut some other way), Spring WebFlux automatically cancels the underlying
pipeline — stopping any further work that would just be wasted effort for
a client who's no longer listening.

## Simple Example

```java
@GetMapping(value = "/live-feed", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> liveFeed() {
    return Flux.interval(Duration.ofSeconds(1))
        .doOnCancel(() -> System.out.println("Client disconnected - stopping feed"))
        .map(tick -> "Update #" + tick);
}
```

If someone opens this endpoint in their browser and then closes the tab,
"Client disconnected - stopping feed" prints automatically — WebFlux
carries the connection close all the way down through the pipeline as a
cancel signal, including shutting off the `Flux.interval()` ticker.

## Why It Matters

In a traditional blocking server, a similar situation (a slow client, or
one that disconnects early) might leave the server-side thread and its
work running anyway until it naturally finishes — burning resources on
work nobody will ever get. WebFlux's automatic cancellation means
abandoned work gets cleaned up right away, which really matters for
long-running or infinite streams, like SSE feeds or big file downloads.
