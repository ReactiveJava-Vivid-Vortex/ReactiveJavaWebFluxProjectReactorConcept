# Performance & Best Practices — Topic Overview

## What Is This Topic About? (In Simple Terms)

This final Project Reactor topic pulls together everything from earlier topics
(threading, backpressure, operators) into a practical checklist for writing
reactive code that actually performs well in production — not just code that
compiles and "looks reactive."

The single most important rule, repeated because it's the #1 real-world bug: **never
block a thread meant to be non-blocking.** A hidden blocking call (legacy JDBC,
`Thread.sleep()`, a blocking HTTP client) sitting on an event-loop or `parallel()`
thread can silently stall every other concurrent request sharing that small pool —
often invisible in low-traffic testing, catastrophic under real load.

```java
// BAD — blocks a precious thread
Mono.fromCallable(() -> { Thread.sleep(1000); return "done"; });

// GOOD — isolates the unavoidable blocking call
Mono.fromCallable(() -> { Thread.sleep(1000); return "done"; })
    .subscribeOn(Schedulers.boundedElastic());

// BEST — avoid blocking entirely with a genuinely non-blocking API
Mono.delay(Duration.ofSeconds(1)).map(t -> "done");
```

A second, less obvious lesson: reactive programming's memory efficiency isn't
automatic. Operators like `.collectList()`, `.distinct()`, and unbounded
`onBackpressureBuffer()` can still accumulate unbounded state if used carelessly on
huge or infinite streams — knowing which operators hold data in memory (and bounding
them) matters just as much as avoiding blocking calls.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Non-blocking execution** | Rule #1: never block an event-loop/parallel() thread — isolate unavoidable blocking calls on `boundedElastic()`. |
| 2 | **Thread utilization** | Reactive's core win: threads stay busy servicing whatever's ready, instead of freezing on I/O waits. |
| 3 | **Scalability** | Same hardware handles far more concurrent I/O-bound requests — the main practical benefit of going reactive. |
| 4 | **Efficient resource usage** | Stream data through the pipeline incrementally instead of buffering it all upfront. |
| 5 | **Memory considerations** | Watch operators that hold state in memory (`collectList`, `distinct`, unbounded buffers) — bound them explicitly. |
| 6 | **Operator selection** | Use `map()` (not `flatMap()`) for sync work; bound `flatMap()` concurrency; use `concatMap()` when order matters. |
| 7 | **Avoiding blocking calls** | Systematically audit for hidden blocking code (legacy JDBC, `RestTemplate`, `Thread.sleep()`, sync file I/O). |

## How It All Fits Together

```
Is there ANY blocking call in this pipeline?
   │
   ├── YES ──▶ isolate on Schedulers.boundedElastic() (or replace with a non-blocking API)
   │
   └── NO — pipeline is genuinely non-blocking
              │
              ▼
        Are you accidentally buffering unbounded data?
        (collectList/distinct/unbounded onBackpressureBuffer on a huge/infinite stream)
              │
              ├── YES ──▶ bound it: buffer(n), limitRate(), bounded onBackpressureBuffer(n)
              └── NO  ──▶ you're following reactive best practices
```

This topic is really a "final exam" checklist: go back through your own reactive
code and ask these two questions — "is anything secretly blocking?" and "is
anything secretly unbounded in memory?" — for every pipeline you write.
