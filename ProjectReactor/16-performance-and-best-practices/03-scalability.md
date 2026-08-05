# Scalability

## In Simple Terms

"Scalability" is a system's ability to handle increasing load (more users, more
requests) without a proportional increase in resources (CPU, memory, threads).
Reactive programming primarily improves **vertical scalability** — squeezing more
concurrent capacity out of the same hardware — by minimizing wasted threads during
I/O waits.

## Simple Example

Rough illustrative comparison for an I/O-heavy workload:

```
Blocking (thread-per-request):
  10,000 concurrent requests -> ~10,000 threads needed -> several GB just for stacks
  -> server likely runs out of resources before 10,000 is even reached

Reactive (non-blocking):
  10,000 concurrent requests -> ~8-16 event-loop threads handle all of them
  -> memory and thread overhead stays roughly constant regardless of concurrency
```

## Why It Matters

Scalability gains from reactive programming are most dramatic for **I/O-bound,
high-concurrency** workloads (many slow external calls, lots of simultaneous
clients) — that's precisely the profile of most modern microservices and public
APIs, which is why reactive frameworks like Spring WebFlux have become popular for
exactly these kinds of systems.
