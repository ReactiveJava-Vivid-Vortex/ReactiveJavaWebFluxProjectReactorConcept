# Elastic

## In Simple Terms

"Elastic" means a system stays responsive across a **wide range of load** — handling
a sudden spike in traffic without needing a proportional increase in resources, and
scaling back down when load decreases. Reactive systems achieve elasticity partly
through efficient thread/resource usage (see [[why-reactive-programming-exists]]).

## Simple Example

A traditional blocking server under load spikes:

```
Normal load:  200 concurrent requests -> 200 threads -> fine
Traffic spike: 5,000 concurrent requests -> needs 5,000 threads -> resource exhaustion,
               requests start failing or queuing indefinitely
```

A reactive, non-blocking server under the same spike:

```
Normal load:  200 concurrent requests -> handled by ~8-16 event-loop threads
Traffic spike: 5,000 concurrent requests -> STILL handled by the same ~8-16 threads
              -> more requests are simply interleaved through non-blocking I/O
```

## Why It Matters

Elasticity is why reactive systems (and Spring WebFlux specifically) are often
chosen for public-facing APIs and services with unpredictable, spiky traffic
patterns — they can absorb sudden load increases far more gracefully than
thread-per-request blocking servers, without needing to provision dramatically more
hardware for peak traffic.
