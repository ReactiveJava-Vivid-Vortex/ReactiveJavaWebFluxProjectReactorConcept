# HTTP/2

## In Simple Terms

HTTP/2 is a major overhaul of the HTTP protocol, built to fix several
performance issues with HTTP/1.1: it brings multiplexing (see
[[multiplexing]]), header compression (HPACK), a more efficient binary
format (instead of plain text), and optional server push — all while still
behaving the same way HTTP/1.1 does (same methods, status codes, headers).

## Simple Example

Enabling HTTP/2 support in Spring WebFlux (built on Netty):

```yaml
server:
  port: 8443
  http2:
    enabled: true
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: changeit
    key-store-type: PKCS12
```

Checking which protocol got used, from a client:

```bash
curl -v --http2 https://localhost:8443/api/products
# Look for "HTTP/2 200" in the output, confirming HTTP/2 was negotiated
```

## Why It Matters

HTTP/2 pairs naturally with reactive, non-blocking servers like WebFlux —
both are built around handling lots of concurrent work efficiently. Put
HTTP/2's connection efficiency together with WebFlux's thread efficiency,
and the performance and scalability benefits stack for high-concurrency
APIs.
