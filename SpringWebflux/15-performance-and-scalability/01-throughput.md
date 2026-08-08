# Throughput

## In Simple Terms

"Throughput" is how many requests a system can get through per unit of
time (like requests per second). This is the main metric where WebFlux
tends to shine over traditional Spring MVC, especially under
high-concurrency, I/O-heavy workloads.

## Simple Example

Illustrative benchmark comparison for an I/O-bound endpoint (calling a
downstream service with ~50ms latency), under high concurrent load:

```
Spring MVC (thread-per-request, 200 threads):
  Throughput plateaus once all 200 threads are busy waiting on the downstream call
  -> roughly (200 threads / 0.05s per request) = ~4,000 requests/sec ceiling

Spring WebFlux (event-loop, non-blocking):
  Threads are never blocked waiting -> can service far more concurrent in-flight
  requests with the same thread count -> throughput scales much higher before
  hitting a different bottleneck (e.g., the downstream service itself)
```

## Why It Matters

Throughput is the number that actually matters for capacity planning —
"how many requests per second can this service take before it falls
over?" Knowing WebFlux's throughput edge is biggest for I/O-bound work
(not CPU-bound) helps you set realistic expectations when benchmarking and
comparing architectures.
