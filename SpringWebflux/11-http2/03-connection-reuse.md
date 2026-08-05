# Connection Reuse

## In Simple Terms

"Connection reuse" means using the same established TCP connection for multiple
requests/responses over time, instead of opening a new connection for every single
request. HTTP/2's multiplexing takes this further than HTTP/1.1's "keep-alive" —
allowing many *concurrent* requests over one reused connection, not just sequential
ones.

## Simple Example

```
HTTP/1.1 with Keep-Alive (sequential reuse):
  Connection stays open, but requests must be sent one at a time, in order
  Request A -> Response A -> Request B -> Response B (same connection, but sequential)

HTTP/2 (concurrent reuse via multiplexing):
  Connection stays open AND handles multiple requests simultaneously
  Request A, Request B sent together -> Response A, Response B interleaved
```

Configuring `WebClient` to reuse connections efficiently via connection pooling
(covered in depth in [[http-connection-pooling]]):

```java
ConnectionProvider provider = ConnectionProvider.builder("custom")
    .maxConnections(50)
    .build();

WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(HttpClient.create(provider)))
    .build();
```

## Why It Matters

Efficient connection reuse (especially combined with HTTP/2 multiplexing)
dramatically reduces the overhead of establishing new TCP/TLS connections
(handshakes are relatively expensive) — critical for high-throughput
service-to-service communication where the same downstream services are called
repeatedly.
