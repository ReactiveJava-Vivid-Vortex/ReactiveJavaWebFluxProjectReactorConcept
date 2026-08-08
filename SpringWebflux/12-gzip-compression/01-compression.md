# Compression

## In Simple Terms

HTTP compression shrinks a response body (usually with gzip) before
sending it over the network, and the client unpacks it once it arrives.
You trade a small bit of CPU time (to compress and decompress) for a much
smaller amount of data actually going over the wire — especially effective
for text-heavy formats like JSON.

## Simple Example

Enabling compression in a Spring WebFlux (Netty-based) application:

```yaml
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/plain
    min-response-size: 1024 # only compress responses larger than 1KB
```

With this on, a client that sends `Accept-Encoding: gzip` (which browsers
do by default) gets back a compressed response, with `Content-Encoding: gzip`
in the headers.

## Why It Matters

For JSON-heavy APIs, gzip compression can shrink response sizes a lot —
often 60-80% smaller for repetitive, text-based JSON — a real win for
bandwidth and how fast things feel, especially for clients on slower
connections like mobile devices.
