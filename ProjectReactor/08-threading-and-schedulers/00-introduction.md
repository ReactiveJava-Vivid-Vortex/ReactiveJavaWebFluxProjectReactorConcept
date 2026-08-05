# Threading & Schedulers — Topic Overview

## What Is This Topic About? (In Simple Terms)

Here's a surprising truth that trips up almost every beginner: **by default, a
Reactor pipeline runs entirely on whatever thread called `.subscribe()`** —
reactive does NOT automatically mean multi-threaded or parallel. Nothing switches
threads unless you explicitly tell it to.

A `Scheduler` is just an abstraction over a pool of threads. You pick one suited to
your workload, and Reactor runs your code on it:

- **`Schedulers.parallel()`** — a small, fixed pool (sized to CPU cores) for
  **CPU-bound, non-blocking** work.
- **`Schedulers.boundedElastic()`** — a large, growable pool for **blocking/I/O**
  work you can't avoid (legacy JDBC, `Thread.sleep()`, file I/O).
- **`Schedulers.single()`** — one dedicated thread, for sequential work.

You control *where* a thread switch happens with two operators that are easy to
confuse:

```java
Mono.fromCallable(() -> blockingLegacyCall())
    .subscribeOn(Schedulers.boundedElastic()) // affects the WHOLE chain, from the source
    .map(this::transform)                     // .publishOn() would only affect what's AFTER it
    .subscribe();
```

The single most important rule in this entire topic: **never block a thread meant
to be non-blocking** (an event-loop or `parallel()` thread). A blocking call
accidentally left there can silently stall every other request sharing that small
thread pool — the #1 real-world reactive performance bug.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Default thread model** | No automatic thread switching — everything runs on the subscribing thread unless told otherwise. |
| 2 | **Thread switching** | Moving execution to a different thread mid-pipeline, via `publishOn()`/`subscribeOn()`. |
| 3 | **Scheduler concept** | An abstraction over a thread pool; pick the right one for your workload instead of managing raw threads. |
| 4 | **boundedElastic()** | Large, growable pool for unavoidable blocking/I/O calls — isolates the damage from event-loop threads. |
| 5 | **parallel()** | Small, fixed pool (CPU-core sized) for genuinely CPU-bound, non-blocking work — never for blocking calls. |
| 6 | **single()** | One dedicated, reusable thread — for sequential, ordered, low-frequency work. |
| 7 | **immediate()** | A no-op scheduler that runs synchronously on the current thread — mostly a default placeholder. |
| 8 | **publishOn()** | Switches threads for everything **downstream** of this point — can be used multiple times in a chain. |
| 9 | **subscribeOn()** | Switches the thread for the **entire chain from the source** — only the first one in a chain has effect. |
| 10 | **Thread pools** | Reusable worker threads avoiding the cost of creating/destroying a thread per task. |
| 11 | **CPU-bound work** | Keeps the processor busy the whole time (hashing, sorting) — scale with `parallel()` + more cores. |
| 12 | **I/O-bound work** | Mostly waiting on network/disk — the exact case reactive/non-blocking I/O was built to optimize. |
| 13 | **Thread affinity** | Not guaranteed in reactive pipelines — code before/after an async boundary may run on different threads (avoid `ThreadLocal`!). |
| 14 | **Scheduler best practices** | Never block event-loop/`parallel()` threads; isolate blocking calls on `boundedElastic()`; minimize thread switches. |

## How It All Fits Together

```
Is the work blocking (JDBC, sleep, file I/O)?
   │
   ├── YES ──▶ subscribeOn(Schedulers.boundedElastic())
   │
   └── NO, is it CPU-heavy (hashing, sorting)?
              │
              ├── YES ──▶ publishOn(Schedulers.parallel())
              │
              └── NO  ──▶ leave it alone — non-blocking I/O
                            (WebClient, R2DBC) needs no scheduler switch at all
```

Golden rule to repeat until it's automatic: **"Is this call blocking? If yes, does
it run on `boundedElastic()`?"** Answering that correctly for every external call in
your codebase is the single biggest factor in whether a reactive app actually
performs well under load.
