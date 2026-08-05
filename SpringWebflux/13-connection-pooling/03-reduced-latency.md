# Reduced Latency

## In Simple Terms

Reusing pooled connections avoids the latency cost of establishing a fresh TCP/TLS
connection for every request — a cost that can easily dominate total request time
for small, fast API calls, especially over higher-latency networks (cross-region
calls, for instance).

## Simple Example

Illustrative comparison for a request to a downstream service on a network with
50ms round-trip latency:

```
Without connection pooling (new connection every request):
  TCP handshake: ~50ms
  TLS handshake: ~50-100ms (additional round trips)
  Actual request/response: ~20ms
  TOTAL: ~120-170ms per request

With connection pooling (connection reused):
  Actual request/response: ~20ms
  TOTAL: ~20ms per request (after the first connection is established)
```

For a service making hundreds of calls per second to the same downstream API, this
difference compounds into a very significant overall latency and throughput
improvement.

## Why It Matters

For latency-sensitive systems (especially those making frequent calls across
network boundaries, like cross-region microservices), connection pooling isn't just
a minor optimization — it can be the difference between an API that feels
instantaneous and one that feels sluggish under load.
