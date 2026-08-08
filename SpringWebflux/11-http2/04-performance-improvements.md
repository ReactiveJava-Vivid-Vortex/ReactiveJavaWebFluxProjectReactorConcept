# Performance Improvements (HTTP/2)

## In Simple Terms

On top of multiplexing, HTTP/2 brings a few other performance wins over
HTTP/1.1: header compression (HPACK, which cuts down repeated header
overhead across many requests), a binary format (quicker to parse than
HTTP/1.1's plain text), and optional server push (though it's mostly fallen
out of favor in practice).

## Simple Example

Header compression example — HTTP/1.1 sends full headers on every request:

```
Request 1: Host: api.example.com, User-Agent: ..., Accept: application/json, ...
Request 2: Host: api.example.com, User-Agent: ..., Accept: application/json, ... (repeated!)
```

HTTP/2's HPACK compresses repeated header data across the connection, so
later requests don't need to resend identical header values in full.

Measuring the practical impact (rough, illustrative comparison for a
header-heavy, high-request-count API):

```
HTTP/1.1: significant overhead from repeated headers + connection limits per host
HTTP/2:   compressed headers + single multiplexed connection
          -> noticeably lower latency and bandwidth usage under high request volume
```

## Why It Matters

For APIs handling lots of small, frequent requests (common with
microservices and mobile clients), HTTP/2's combined perks (multiplexing
plus header compression) can meaningfully cut both latency and bandwidth
use — and stack nicely with WebFlux's own efficiency gains from
non-blocking I/O.
