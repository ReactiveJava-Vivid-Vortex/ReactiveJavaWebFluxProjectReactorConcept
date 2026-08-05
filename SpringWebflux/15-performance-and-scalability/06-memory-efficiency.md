# Memory Efficiency

## In Simple Terms

WebFlux applications can achieve significantly better memory efficiency than
traditional blocking servers under high concurrency — both because far fewer
threads are needed (each thread's stack consumes real memory, see
[[ram-and-memory-model]] in the ProjectReactor notes), and because streaming
avoids fully buffering large datasets in memory.

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

Combined with streaming (avoiding `.collectList()` on huge datasets, see
[[memory-efficient-processing]]), a WebFlux application's memory footprint stays far
more predictable and bounded, even as concurrent load and dataset sizes grow.

## Why It Matters

Better memory efficiency directly translates to real infrastructure cost savings —
handling the same traffic with less RAM per server instance, or handling more
traffic with the same hardware — a concrete, measurable business benefit of
adopting the reactive model for the right (I/O-heavy, high-concurrency) workloads.
