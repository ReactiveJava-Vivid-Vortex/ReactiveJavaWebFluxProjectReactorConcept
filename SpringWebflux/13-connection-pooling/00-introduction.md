# Connection Pooling — Topic Overview

## What Is This Topic About? (In Simple Terms)

Every new TCP (and TLS, if HTTPS) connection requires a network round-trip
handshake before any actual data can flow — a real, measurable cost, especially over
higher-latency networks. **Connection pooling** avoids paying that cost repeatedly
by keeping a set of already-established connections open and reusing them for
subsequent calls to the same host, instead of opening and closing a fresh connection
every single time.

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

The real skill in this topic isn't just "turn pooling on" (it's on by default for
`WebClient`) — it's **sizing it correctly** for your traffic. Too small a pool
causes requests to queue and wait for a free connection, adding latency; too large a
pool can overwhelm the downstream service with more concurrent connections than it
can handle. Tune based on real, observed traffic and monitored pool metrics, not
guesswork.

The payoff is concrete: for a downstream call over a network with meaningful
latency, skipping the repeated handshake cost can turn a ~150ms request into a
~20ms one — a difference that compounds dramatically at high request volumes.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **HTTP Connection Pooling** | Reuse already-established connections instead of paying a fresh TCP/TLS handshake cost every call. |
| 2 | **Efficient client connections** | Size the pool (max connections, idle timeout) to match real observed traffic — monitor, don't guess. |
| 3 | **Reduced latency** | Skipping repeated handshakes can be the difference between ~150ms and ~20ms per call, especially cross-region. |

## How It All Fits Together

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

Pair this with the HTTP/2 topic for maximum effect: HTTP/2's multiplexing means
even fewer pooled connections are needed to serve high concurrent request volume to
the same host.
