# Memory Efficiency

## In Simple Terms

WebFlux apps can be a lot more memory-efficient than traditional blocking
servers under heavy concurrency — both because they need far fewer
threads (each thread's stack eats real memory, see
[[ram-and-memory-model]] in the Project Reactor notes), and because
streaming avoids fully loading big datasets into memory.

## Simple Example

Rough illustrative comparison, 10,000 concurrent slow requests:

```
Spring MVC (thread-per-request):
  ~10,000 threads needed (if the server could even support that many)
  ~1MB stack per thread -> ~10GB just for thread stacks alone

Spring WebFlux (event-loop):
  ~8-16 threads needed regardless of concurrent request count
  ~8-16 MB for thread stacks total -- negligible by comparison
```

Combine that with streaming (avoiding `.collectList()` on huge datasets,
see [[memory-efficient-processing]]), and a WebFlux app's memory footprint
stays far more predictable and bounded, even as traffic and dataset sizes
grow.

## Why It Matters

Better memory efficiency translates directly into real infrastructure
savings — handling the same traffic with less RAM per server, or more
traffic with the same hardware — a concrete, measurable payoff of going
reactive for the right (I/O-heavy, high-concurrency) workloads.
