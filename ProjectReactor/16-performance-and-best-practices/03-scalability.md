# Scalability

## In Simple Terms

"Scalability" is how well a system handles more load — more users, more
requests — without needing a proportional pile of extra resources (CPU,
memory, threads). Reactive programming mostly helps by squeezing more
capacity out of the same hardware, by wasting far fewer threads while
waiting on slow I/O.

## Simple Example

Rough, illustrative comparison for an I/O-heavy workload:

```
Blocking (thread-per-request):
  10,000 concurrent requests -> ~10,000 threads needed -> several GB just for stacks
  -> server likely runs out of resources before 10,000 is even reached

Reactive (non-blocking):
  10,000 concurrent requests -> ~8-16 event-loop threads handle all of them
  -> memory and thread overhead stays roughly constant regardless of concurrency
```

## Why It Matters

The scalability gains from reactive programming really show up in
I/O-heavy, high-concurrency workloads — lots of slow external calls, lots
of clients connected at once. That's exactly the shape of most modern
microservices and public APIs, which is why frameworks like Spring WebFlux
took off for these kinds of systems in the first place.
