# Why WebFlux Scales Better Than Spring MVC

## In Simple Terms

Spring MVC uses a **thread-per-request** model built on the Servlet API: each
incoming HTTP request is handled by a dedicated thread from a pool (often sized
around 200 by default), which blocks for the entire duration of any I/O the request
performs. Spring WebFlux uses a small, fixed number of **event-loop threads** (via
Netty) that never block — they're always free to service whichever request has data
ready.

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

This difference is most dramatic for **I/O-bound, high-concurrency** workloads —
exactly the profile of most public APIs and microservices. For CPU-bound workloads
with modest concurrency, Spring MVC often performs just as well (or is simpler to
reason about), which is why choosing WebFlux should be a deliberate decision based on
your actual traffic profile, not a default choice for every project.
