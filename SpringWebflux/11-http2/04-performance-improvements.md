# Performance Improvements (HTTP/2)

## In Simple Terms

Beyond multiplexing, HTTP/2 brings several other performance improvements over
HTTP/1.1: **header compression** (HPACK, reducing repetitive header overhead across
many requests), **binary framing** (more efficient to parse than HTTP/1.1's
text-based format), and optional **server push** (though largely deprecated in
practice in favor of other techniques).

## Simple Example

Header compression example — HTTP/1.1 sends full headers on every request:

```
Request 1: Host: api.example.com, User-Agent: ..., Accept: application/json, ...
Request 2: Host: api.example.com, User-Agent: ..., Accept: application/json, ... (repeated!)
```

HTTP/2's HPACK compresses repeated header data across the connection, so subsequent
requests don't need to resend identical header values in full.

Measuring the practical impact (rough illustrative comparison for a
header-heavy, high-request-count API):

```
HTTP/1.1: significant overhead from repeated headers + connection limits per host
HTTP/2:   compressed headers + single multiplexed connection
          -> noticeably lower latency and bandwidth usage under high request volume
```

## Why It Matters

For APIs with many small, frequent requests (a common pattern in microservices and
mobile clients), HTTP/2's combined improvements (multiplexing + header compression)
can meaningfully reduce both latency and bandwidth usage — compounding well with
WebFlux's own efficiency gains from non-blocking I/O.
