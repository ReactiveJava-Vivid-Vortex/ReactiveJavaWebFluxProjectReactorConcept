# Efficient Client Connections

## In Simple Terms

Beyond just pooling, "efficient client connections" means correctly sizing your
pool and configuring its behavior (idle timeouts, max connections, queuing behavior)
to match your actual traffic patterns — too small a pool causes requests to queue
and wait; too large wastes resources and can overwhelm the downstream service.

## Simple Example

```java
ConnectionProvider provider = ConnectionProvider.builder("api-pool")
    .maxConnections(50)                          // tuned to downstream capacity
    .pendingAcquireTimeout(Duration.ofSeconds(5)) // fail fast if pool is exhausted
    .maxIdleTime(Duration.ofSeconds(60))
    .evictInBackground(Duration.ofSeconds(120))    // periodic cleanup of stale connections
    .build();
```

Monitoring pool health (via Micrometer metrics, if configured) helps validate
whether the pool size is well-tuned:

```
reactor.netty.connection.provider.active.connections
reactor.netty.connection.provider.pending.connections
```

## Why It Matters

An under-sized pool causes requests to queue and experience added latency waiting
for a free connection; an over-sized pool can overwhelm a downstream service with
more concurrent connections than it can handle. Tuning pool size based on real,
observed traffic (rather than guessing) is essential for reliable, efficient
service-to-service communication at scale.
