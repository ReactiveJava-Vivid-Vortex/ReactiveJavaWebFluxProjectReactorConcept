# Multiplexing

## In Simple Terms

Multiplexing is HTTP/2's trick of sending several independent requests and
responses over one single TCP connection at the same time, without any of
them blocking each other. In HTTP/1.1, a connection could really only
handle one request at a time, so browsers had to open several separate
connections just to load things in parallel.

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

Multiplexing cuts down connection overhead a lot — fewer TCP connections
means less resource use on both ends, faster page loads (no more "waiting
for a free connection slot"), and better overall network efficiency,
especially valuable for API-heavy apps making lots of small concurrent
calls.
