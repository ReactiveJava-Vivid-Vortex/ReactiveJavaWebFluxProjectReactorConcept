# Reduced Latency

## In Simple Terms

Reusing pooled connections skips the latency cost of setting up a fresh
TCP/TLS connection for every single request — a cost that can easily
dominate the total time for small, fast API calls, especially over
higher-latency networks (cross-region calls, for example).

## Simple Example

Illustrative comparison for a request to a downstream service on a network
with 50ms round-trip latency:

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

For a service making hundreds of calls per second to the same downstream
API, this difference adds up into a very real improvement in overall
latency and throughput.

## Why It Matters

For latency-sensitive systems — especially ones making frequent calls
across network boundaries, like cross-region microservices — connection
pooling isn't just a minor tweak. It can be the difference between an API
that feels instant and one that feels sluggish under load.
