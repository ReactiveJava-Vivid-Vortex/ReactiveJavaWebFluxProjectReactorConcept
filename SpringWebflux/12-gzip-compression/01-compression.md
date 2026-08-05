# Compression

## In Simple Terms

HTTP compression shrinks the response body (typically using gzip) before sending it
over the network, and the client decompresses it upon arrival. This trades a small
amount of CPU time (to compress/decompress) for a significant reduction in the
amount of data transferred — especially effective for text-based formats like JSON.

## Simple Example

Enabling compression in a Spring WebFlux (Netty-based) application:

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/plain
    min-response-size: 1024 # only compress responses larger than 1KB
```

With this enabled, a client sending `Accept-Encoding: gzip` (which browsers do by
default) receives a compressed response, with `Content-Encoding: gzip` in the
response headers.

## Why It Matters

For JSON-heavy APIs, gzip compression can reduce response sizes dramatically (often
60-80% smaller for repetitive, text-based JSON) — a substantial win for bandwidth
usage and perceived latency, especially for clients on slower network connections
(mobile devices, for example).
