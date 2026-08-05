# Performance & Scalability — Topic Overview

## What Is This Topic About? (In Simple Terms)

This topic ties together everything you've learned into a concrete performance
story: what actually makes a WebFlux application faster/more scalable than the
equivalent Spring MVC app, and what could quietly ruin those benefits.

The headline metric is **throughput** (requests handled per second under load) —
where WebFlux's advantage is most dramatic for **I/O-bound, high-concurrency**
workloads. A blocking server's throughput plateaus once its thread pool is fully
occupied waiting on slow I/O; a WebFlux server's threads are never blocked waiting,
so it can absorb far more concurrent in-flight requests before hitting a different
bottleneck.

```
Spring MVC (200 threads, 50ms downstream call):  throughput ceiling ≈ 4,000 req/sec
Spring WebFlux (8-16 threads, same call):         throughput scales much higher,
                                                    since threads never sit idle waiting
```

But this all depends on one non-negotiable condition, repeated from earlier topics:
**every part of the pipeline must stay genuinely non-blocking** — controller,
service, repository (R2DBC, not JDBC), and external calls (WebClient, not
`RestTemplate`). A single hidden blocking call anywhere undoes the whole benefit for
every request sharing that thread.

The other side of the coin is **resource efficiency**: fewer threads needed means
dramatically less memory consumed by thread stacks alone (potentially gigabytes
saved at high concurrency) — a real, measurable infrastructure cost benefit, not
just an abstract performance number.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Throughput** | Requests/sec a system handles — WebFlux's advantage is biggest for I/O-heavy, high-concurrency workloads. |
| 2 | **Non-blocking execution** | The non-negotiable condition: EVERY layer (controller/service/repo/external calls) must stay non-blocking. |
| 3 | **Resource utilization** | High utilization = threads almost always doing real work, not idly waiting — monitor thread counts to verify. |
| 4 | **Scalability** | Same hardware absorbs far more concurrent slow requests — the core practical value proposition of WebFlux. |
| 5 | **Backpressure** | Automatically paces streaming responses to match client/network speed — prevents server memory overload. |
| 6 | **Memory efficiency** | Fewer threads (smaller stacks) + streaming (no full buffering) = far lower memory footprint at scale. |

## How It All Fits Together

```
High-concurrency, I/O-heavy workload?
   │
   ├── YES ──▶ WebFlux's scalability advantage applies strongly
   │              │
   │              ▼
   │        Verify EVERY layer is non-blocking (R2DBC, WebClient, no hidden JDBC/RestTemplate)
   │              │
   │              ▼
   │        Monitor thread count + memory — should stay small & stable under rising load
   │
   └── NO (CPU-bound, low concurrency) ──▶ WebFlux's benefit here is minimal — Spring MVC may be simpler
```

This topic is the "does it actually work in practice?" checkpoint — the theoretical
scalability promised by earlier topics only materializes if you've actually kept
every layer non-blocking, which is worth re-verifying whenever performance doesn't
match expectations.
