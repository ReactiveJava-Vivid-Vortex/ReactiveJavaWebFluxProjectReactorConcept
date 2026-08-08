# HTTP Connection Pooling

## In Simple Terms

Opening a new TCP (and TLS, for HTTPS) connection isn't free — it takes a
network round-trip handshake before any actual data can flow. Connection
pooling avoids paying that cost over and over by keeping a set of
already-open connections around and reusing them for later requests to the
same host, instead of tearing one down and building a new one every time.

## Simple Example

Configuring a connection pool for `WebClient` (built on Reactor Netty):

```java
ConnectionProvider provider = ConnectionProvider.builder("my-pool")
    .maxConnections(100)               // max concurrent connections
    .pendingAcquireMaxCount(500)        // max requests queued waiting for a connection
    .maxIdleTime(Duration.ofSeconds(30)) // close idle connections after 30s
    .build();

HttpClient httpClient = HttpClient.create(provider);

WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(httpClient))
    .baseUrl("https://api.example.com")
    .build();
```

## Why It Matters

Without connection pooling, every single outgoing `WebClient` call would
pay the full price of a new TCP/TLS handshake — real, unnecessary overhead
when you're calling the same downstream service often. Pooling spreads
that cost across many requests, which really matters for high-throughput
service-to-service communication.
