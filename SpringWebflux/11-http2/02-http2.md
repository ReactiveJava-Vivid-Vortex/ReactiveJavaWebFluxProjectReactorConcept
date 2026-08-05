# HTTP/2

## In Simple Terms

**HTTP/2** is a major revision of the HTTP protocol, designed to fix several
performance limitations of HTTP/1.1: it introduces multiplexing (see
[[multiplexing]]), header compression (HPACK), binary framing (instead of
human-readable text), and server push — all while remaining semantically compatible
with HTTP/1.1 (same methods, status codes, headers).

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

Verifying the protocol used, from a client:

```bash
curl -v --http2 https://localhost:8443/api/products
# Look for "HTTP/2 200" in the output, confirming HTTP/2 was negotiated
```

## Why It Matters

HTTP/2 pairs particularly well with reactive, non-blocking servers like WebFlux —
both are designed around efficiently handling many concurrent operations with
minimal overhead. Combining HTTP/2's connection efficiency with WebFlux's thread
efficiency compounds the overall performance and scalability benefits for
high-concurrency APIs.
