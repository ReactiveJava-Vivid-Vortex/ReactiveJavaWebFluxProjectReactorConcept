# Backpressure

## In Simple Terms

Backpressure lets a slow consumer (e.g., a client with a slow network connection)
control how fast a fast producer (e.g., a server streaming a huge file) sends it
data — preventing the consumer from being overwhelmed with more data than it can
handle at once.

## Simple Example

In a Spring WebFlux streaming endpoint:

```java
@GetMapping(value = "/large-dataset", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<Record> streamRecords() {
    return recordRepository.findAll(); // Flux<Record>, potentially millions of rows
}
```

If the HTTP client (or its network connection) can only consume data slowly, the
underlying Netty/Reactor infrastructure naturally applies backpressure — the
database query is paced to only fetch as much data as the client can currently
absorb, rather than loading everything into server memory upfront and hoping the
client keeps up.

## Why It Matters

Without backpressure, a fast server could stream data faster than a slow client (or
network) can receive it, causing the server to buffer huge amounts of unsent data in
memory. Backpressure — baked into the Reactive Streams specification that WebFlux is
built on — ensures memory usage stays bounded and proportional to actual throughput,
even when producer and consumer speeds differ significantly.
