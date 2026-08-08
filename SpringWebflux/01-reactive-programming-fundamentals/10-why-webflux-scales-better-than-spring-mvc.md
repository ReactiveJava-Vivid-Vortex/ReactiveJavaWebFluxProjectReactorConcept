# Why WebFlux Scales Better Than Spring MVC

## In Simple Terms

Spring MVC hands each incoming request its own dedicated thread from a
pool (often around 200 by default) — and that thread freezes for however
long the request's I/O takes. Spring WebFlux instead uses a small, fixed
handful of event-loop threads (via Netty) that never freeze — they're
always free to service whichever request happens to have data ready.

## Simple Example

```
Spring MVC (thread-per-request), 200-thread pool:
  200 concurrent slow requests (each waiting 500ms on a DB call)
  -> all 200 threads are busy/blocked
  -> request #201 must WAIT in queue, even though the CPU itself is mostly idle

Spring WebFlux (event-loop), ~8-16 threads:
  10,000 concurrent slow requests (each waiting 500ms on a DB call)
  -> the small pool of threads is never blocked waiting
  -> they're constantly cycling through servicing whichever request's data
     has become ready, so far more concurrent requests can be in-flight
     simultaneously with the same thread count
```

## Why It Matters

This difference matters most for I/O-heavy workloads with lots of
concurrent traffic — exactly the shape of most public APIs and
microservices. For CPU-heavy workloads with modest concurrency, plain
Spring MVC often does just as well (and is simpler to reason about), which
is why choosing WebFlux should be a deliberate call based on your actual
traffic, not just a default for every project.
