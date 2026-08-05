# Resource Utilization

## In Simple Terms

"Resource utilization" measures how effectively a system uses its available CPU,
memory, and thread resources. A well-built WebFlux application achieves high
utilization — nearly all its threads are doing productive work most of the time,
rather than sitting idle waiting on I/O.

## Simple Example

Monitoring resource utilization via actuator/metrics endpoints:

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

Key metrics to watch for a WebFlux application:

```
reactor.netty.eventloop.pending.tasks   - tasks waiting for an event-loop thread
jvm.memory.used                          - overall JVM memory usage
jvm.threads.live                         - total thread count (should stay small and stable)
```

A well-behaved WebFlux app under load should show a **stable, small** thread count
even as concurrent request volume rises significantly — a sign that non-blocking I/O
is working as intended, rather than threads accumulating due to hidden blocking calls.

## Why It Matters

Actively monitoring resource utilization (rather than assuming reactive code is
automatically efficient) helps catch regressions — like a newly introduced blocking
call causing thread counts to climb unexpectedly under load, which would otherwise
be an easy-to-miss, silent performance regression.
