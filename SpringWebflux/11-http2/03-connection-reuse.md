# Connection Reuse

## In Simple Terms

"Connection reuse" means using the same already-open TCP connection for
several requests over time, instead of opening a fresh one every time.
HTTP/2's multiplexing takes this a step further than HTTP/1.1's
"keep-alive" — it lets many requests run over one reused connection *at
the same time*, not just one after another.

## Simple Example

```
HTTP/1.1 with Keep-Alive (sequential reuse):
  Connection stays open, but requests must be sent one at a time, in order
  Request A -> Response A -> Request B -> Response B (same connection, but sequential)

HTTP/2 (concurrent reuse via multiplexing):
  Connection stays open AND handles multiple requests simultaneously
  Request A, Request B sent together -> Response A, Response B interleaved
```

Setting up `WebClient` to reuse connections efficiently through connection
pooling (covered in depth in [[http-connection-pooling]]):

```java
ConnectionProvider provider = ConnectionProvider.builder("custom")
    .maxConnections(50)
    .build();

WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(HttpClient.create(provider)))
    .build();
```

## Why It Matters

Reusing connections efficiently — especially combined with HTTP/2's
multiplexing — cuts down a lot of the cost of opening new TCP/TLS
connections (handshakes aren't cheap) — important for high-throughput
service-to-service calls where you're repeatedly hitting the same
downstream services.
