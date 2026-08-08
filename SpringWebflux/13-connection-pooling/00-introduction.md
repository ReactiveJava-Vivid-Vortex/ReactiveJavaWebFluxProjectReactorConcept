# Q1. Why Not Just Open a New Connection for Every Call?

## Simple Explanation (Think of Renting a Car Every Trip vs Keeping One in the Driveway)

Establishing a new TCP (+ TLS) connection requires a network round-trip handshake
before any actual data moves — like renting a brand-new car every single time you
need to drive somewhere, even to the same nearby store. **Connection pooling**
keeps a few cars already running in the driveway, ready to reuse instantly.

```java
ConnectionProvider provider = ConnectionProvider.builder("my-pool")
    .maxConnections(100)
    .maxIdleTime(Duration.ofSeconds(30))
    .build();

WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(HttpClient.create(provider)))
    .baseUrl("https://api.example.com")
    .build();
```

This is **on by default** for `WebClient` — the real skill is sizing it correctly,
not just "turning it on."

---

## Q2. How Do I Size the Pool Correctly?

| Pool Size | Consequence |
|---|---|
| Too small | Requests queue, waiting for a free connection — added latency |
| Too large | Can overwhelm the downstream service with more concurrent connections than it can handle |
| Just right | Based on real, observed traffic + monitored pool metrics — not guesswork |

```
reactor.netty.connection.provider.active.connections   ← monitor these
reactor.netty.connection.provider.pending.connections
```

---

## Q3. What's the Concrete Latency Payoff?

```
First call to a host:  new TCP/TLS connection (expensive handshake) ≈ 150ms
Subsequent calls:      REUSE the pooled connection (handshake skipped!) ≈ 20ms
```

Over a network with meaningful latency (cross-region calls, for instance), this
difference compounds dramatically at high request volumes.

---

## Q4. Interview-Style Q&A

### Is connection pooling enabled by default in `WebClient`?

**Yes** — Reactor Netty's `HttpClient` uses a connection pool automatically;
you only need custom configuration if the defaults don't fit your traffic.

### What happens if the pool runs out of available connections?

New requests wait (up to `pendingAcquireTimeout`) for one to free up, or fail if
that timeout is exceeded — which is why sizing correctly matters.

### Does connection pooling pair well with HTTP/2?

**Yes** — HTTP/2's multiplexing means even fewer pooled connections are needed to
serve high concurrent request volume to the same host.

---

## Q5. Summary

```
First call to a host  ──▶  new TCP/TLS connection established (expensive handshake)
                                     │
                                     ▼
                          Connection kept OPEN in the pool
                                     │
Subsequent calls to same host ──▶  REUSE the pooled connection (handshake skipped!)
                                     │
                                     ▼
                    Lower latency + less handshake overhead, compounding
                    across every request made to that downstream service
```

### One sentence to remember

> **"Connection pooling is keeping a car running in the driveway instead of
> renting a new one for every trip — reuse beats re-establishing, every
> time."**
