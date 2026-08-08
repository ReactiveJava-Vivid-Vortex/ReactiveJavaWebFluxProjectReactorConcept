# Resource Utilization

## In Simple Terms

"Resource utilization" measures how well a system uses the CPU, memory,
and threads it has available. A well-built WebFlux app gets high
utilization — nearly all its threads are doing productive work most of the
time, instead of sitting idle waiting on I/O.

## Simple Example

Monitoring resource utilization through actuator/metrics endpoints:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: metrics, health
  metrics:
    tags:
      application: my-webflux-app
```

Key metrics worth watching for a WebFlux app:

```
reactor.netty.eventloop.pending.tasks   - tasks waiting for an event-loop thread
jvm.memory.used                          - overall JVM memory usage
jvm.threads.live                         - total thread count (should stay small and stable)
```

A healthy WebFlux app under load should show a small, stable thread count
even as concurrent traffic climbs significantly — a sign non-blocking I/O
is doing its job, rather than threads piling up because of hidden blocking
calls.

## Why It Matters

Actively watching resource utilization — instead of just assuming reactive
code is automatically efficient — helps you catch regressions early, like
a newly introduced blocking call quietly pushing thread counts up under
load, which would otherwise be an easy thing to miss.
