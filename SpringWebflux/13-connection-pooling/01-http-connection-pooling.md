# HTTP Connection Pooling

## In Simple Terms

Establishing a new TCP (and TLS, if HTTPS) connection is relatively expensive — it
involves a network round-trip handshake before any actual data can be sent.
**Connection pooling** avoids this cost by keeping a set of already-established
connections open and reusing them for subsequent requests to the same host, rather
than tearing down and re-establishing a connection every time.

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

Without connection pooling, every single outgoing `WebClient` call would pay the
full cost of a new TCP/TLS handshake — a significant, unnecessary overhead when
making frequent calls to the same downstream service. Pooling amortizes that cost
across many requests, which is essential for high-throughput service-to-service
communication.
