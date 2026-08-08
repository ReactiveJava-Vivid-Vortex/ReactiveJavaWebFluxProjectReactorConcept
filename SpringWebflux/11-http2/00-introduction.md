# Q1. Why Does HTTP/2 Pair So Well with WebFlux?

## Simple Explanation (Think of a Single Multi-Lane Highway vs Many Single-Lane Roads)

HTTP/1.1 could really only handle **one request at a time per connection** —
browsers had to open several parallel connections (single-lane roads) just to load
things concurrently. HTTP/2 sends everything over **one multi-lane highway**
(a single connection), all vehicles moving simultaneously.

```
HTTP/1.1: Request A -> Connection 1
          Request B -> Connection 2   (needs MULTIPLE connections for concurrency)
          Request C -> Connection 3

HTTP/2:   Request A, B, C  ──▶  ONE connection, multiplexed, headers compressed
```

Both WebFlux and HTTP/2 chase the exact same goal: handle many concurrent
operations with minimal overhead — one on the thread level, one on the connection
level.

---

## Q2. What Is "Multiplexing," Concretely?

Many requests and responses flow over a **single** TCP connection, simultaneously,
interleaved — instead of needing one connection per concurrent request.

```yaml
server:
  http2:
    enabled: true
  ssl:
    enabled: true   # HTTP/2 in browsers requires TLS
    key-store: classpath:keystore.p12
```

---

## Q3. What Other Improvements Does HTTP/2 Bring?

| Feature | What It Does |
|---|---|
| Multiplexing | Many requests/responses over ONE connection at once |
| Header compression (HPACK) | Avoids resending identical headers on every request |
| Binary framing | More efficient to parse than HTTP/1.1's plain text format |

```
Request 1: Host: api.example.com, User-Agent: ..., Accept: application/json, ...
Request 2: Host: api.example.com, User-Agent: ..., Accept: application/json, ... (repeated in HTTP/1.1!)
```

HTTP/2's HPACK compresses this repeated header data across the connection.

---

## Q4. What's the Practical Payoff?

Fewer TCP connections needed overall means fewer expensive TCP/TLS handshakes —
directly reducing both latency and bandwidth usage, especially valuable for APIs
with many small, frequent requests (a very common microservices/mobile pattern).

---

## Q5. Interview-Style Q&A

### Does enabling HTTP/2 require code changes in my controllers?

**No** — it's purely a server configuration setting (`server.http2.enabled=true`
+ TLS). Your `Mono`/`Flux`-returning handlers don't change at all.

### Does HTTP/2 change HTTP semantics (methods, status codes)?

**No** — it's semantically compatible with HTTP/1.1; only the underlying
transport/framing mechanics change.

### Is HTTP/2 in browsers possible without TLS?

**Effectively no** — browsers require HTTPS for HTTP/2, even though the spec
technically allows unencrypted HTTP/2.

---

## Q6. Summary

```
HTTP/1.1: Request A ──▶ Connection 1
          Request B ──▶ Connection 2   (needs multiple connections for concurrency)
          Request C ──▶ Connection 3

HTTP/2:   Request A, B, C  ──▶  ONE connection, multiplexed, headers compressed
          (fewer handshakes, less overhead, lower latency under load)
```

### One sentence to remember

> **"HTTP/2's multiplexing does for CONNECTIONS what WebFlux's event loop does
> for THREADS — serve far more concurrent work with far less overhead."**
