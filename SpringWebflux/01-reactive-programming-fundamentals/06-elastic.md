# Elastic

## In Simple Terms

"Elastic" means a system stays responsive across a wide range of traffic —
it can absorb a sudden spike without needing a matching pile of extra
resources, and it can scale back down once things calm down. Reactive
systems get this partly from using threads and resources so efficiently
(see [[why-reactive-programming-exists]]).

## Simple Example

A traditional blocking server hitting a traffic spike:

```
Normal load:  200 concurrent requests -> 200 threads -> fine
Traffic spike: 5,000 concurrent requests -> needs 5,000 threads -> resource exhaustion,
               requests start failing or queuing indefinitely
```

A reactive, non-blocking server hitting the same spike:

```
Normal load:  200 concurrent requests -> handled by ~8-16 event-loop threads
Traffic spike: 5,000 concurrent requests -> STILL handled by the same ~8-16 threads
              -> more requests are simply interleaved through non-blocking I/O
```

## Why It Matters

Elasticity is exactly why reactive systems — Spring WebFlux especially —
get picked for public APIs and services with unpredictable, spiky traffic.
They can absorb a sudden rush far more gracefully than a
thread-per-request server, without needing a big pile of extra hardware
just for the peak moments.
