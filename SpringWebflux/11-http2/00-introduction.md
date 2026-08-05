# HTTP/2 — Topic Overview

## What Is This Topic About? (In Simple Terms)

HTTP/2 is a major upgrade to the HTTP protocol, and it pairs unusually well with
WebFlux — both are designed around the same core idea: efficiently handling many
concurrent operations with minimal overhead. The headline feature is
**multiplexing**: HTTP/1.1 could really only handle one request at a time per
connection, forcing browsers to open several parallel connections just to load
things concurrently. HTTP/2 sends many requests and responses over a **single**
connection, simultaneously, interleaved.

```yaml
server:
  http2:
    enabled: true
  ssl:
    enabled: true   # HTTP/2 in browsers requires TLS
    key-store: classpath:keystore.p12
```

Beyond multiplexing, HTTP/2 also brings **header compression** (HPACK — avoiding
resending identical headers on every request) and a more efficient **binary
framing** format (versus HTTP/1.1's plain text). Together, these reduce both
connection overhead and per-request overhead — particularly valuable for APIs with
many small, frequent requests (a very common microservices/mobile pattern).

The practical upshot: fewer TCP connections needed overall (better **connection
reuse**), which reduces the relatively expensive cost of TCP/TLS handshakes, leading
to measurably lower latency and bandwidth usage under high request volume.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Multiplexing** | Many requests/responses over ONE connection at once — HTTP/1.1 needed multiple connections for concurrency. |
| 2 | **HTTP/2** | The protocol bringing multiplexing, header compression, and binary framing — semantically compatible with HTTP/1.1. |
| 3 | **Connection reuse** | HTTP/2 lets one connection serve many *concurrent* requests, not just sequential ones (beyond simple keep-alive). |
| 4 | **Performance improvements** | Combined effect: less connection overhead, less repeated header data, lower latency and bandwidth at scale. |

## How It All Fits Together

```
HTTP/1.1: Request A ──▶ Connection 1
          Request B ──▶ Connection 2   (needs multiple connections for concurrency)
          Request C ──▶ Connection 3

HTTP/2:   Request A, B, C  ──▶  ONE connection, multiplexed, headers compressed
          (fewer handshakes, less overhead, lower latency under load)
```

Enable HTTP/2 alongside WebFlux's own non-blocking efficiency and the gains
compound: fewer wasted threads (WebFlux) + fewer wasted connections (HTTP/2) = a
server that scales significantly better under high-concurrency traffic than either
improvement alone would achieve.
