# Efficient Client Connections

## In Simple Terms

Beyond just having a pool, "efficient client connections" means actually
sizing that pool correctly and configuring its behavior (idle timeouts,
max connections, how it handles waiting requests) to match your real
traffic — too small a pool means requests queue up and wait; too large
wastes resources and can even overwhelm the downstream service.

## Simple Example

```java
ConnectionProvider provider = ConnectionProvider.builder("api-pool")
    .maxConnections(50)                          // tuned to downstream capacity
    .pendingAcquireTimeout(Duration.ofSeconds(5)) // fail fast if pool is exhausted
    .maxIdleTime(Duration.ofSeconds(60))
    .evictInBackground(Duration.ofSeconds(120))    // periodic cleanup of stale connections
    .build();
```

Watching pool health (through Micrometer metrics, if you've set that up)
helps confirm whether your pool size is actually well-tuned:

```
reactor.netty.connection.provider.active.connections
reactor.netty.connection.provider.pending.connections
```

## Why It Matters

Too small a pool means requests sit around waiting for a free connection,
adding latency; too big a pool can throw more concurrent connections at a
downstream service than it can actually handle. Tuning pool size off real,
observed traffic — instead of guessing — is important for reliable,
efficient service-to-service calls at scale.
