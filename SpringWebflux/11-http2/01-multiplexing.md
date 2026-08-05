# Multiplexing

## In Simple Terms

**Multiplexing** is HTTP/2's ability to send multiple independent requests and
responses over a **single TCP connection**, simultaneously, without them blocking
each other. In HTTP/1.1, each connection could effectively only handle one request
at a time (barring pipelining workarounds), so browsers had to open multiple parallel
connections to load resources concurrently.

## Simple Example

```
HTTP/1.1 (without multiplexing):
  Connection 1: Request A -----> Response A
  Connection 2: Request B -----> Response B
  Connection 3: Request C -----> Response C
  (needs 3 separate TCP connections for 3 concurrent requests)

HTTP/2 (with multiplexing):
  Connection 1: Request A, B, C all sent together
                Response A, B, C interleaved on the SAME connection
  (only 1 TCP connection needed for all 3 concurrent requests)
```

Enabling HTTP/2 in a Spring WebFlux application (Netty-based):

```yaml
server:
  http2:
    enabled: true
  ssl:
    enabled: true # HTTP/2 in browsers requires TLS
    key-store: classpath:keystore.p12
    key-store-password: changeit
```

## Why It Matters

Multiplexing reduces connection overhead significantly — fewer TCP connections
means less resource usage on both client and server, faster page loads (no more
"waiting for a free connection slot"), and better overall network efficiency,
especially valuable for API-heavy applications making many small concurrent calls.
