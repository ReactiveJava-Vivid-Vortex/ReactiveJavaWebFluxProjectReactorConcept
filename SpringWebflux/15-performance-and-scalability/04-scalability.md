# Scalability (WebFlux)

## In Simple Terms

As covered in the Project Reactor notes ([[scalability]]), WebFlux's
scalability edge shows up most in I/O-bound, high-concurrency workloads —
the same hardware can handle a lot more concurrent requests than a
thread-per-request blocking model, since threads are never wasted waiting
on I/O.

## Simple Example

A practical illustration of scaling up under load testing:

```
Load test: 5,000 concurrent users, each making a request that involves
           a 100ms downstream database call

Spring MVC (200-thread pool):
  Most threads quickly become occupied waiting on the DB call
  Requests beyond thread capacity queue up, increasing latency significantly

Spring WebFlux (8-16 event-loop threads):
  Threads never block waiting; they cycle through servicing whichever
  request's data becomes ready
  Can absorb far more of the 5,000 concurrent requests without queuing delays
```

## Why It Matters

Seeing scalability as WebFlux's real value — rather than "it's just
faster" — sets the right expectations: WebFlux won't make one single
request faster, but it dramatically changes how many concurrent requests
the same hardware can handle smoothly — which is exactly the problem worth
solving for high-traffic, I/O-heavy APIs.
