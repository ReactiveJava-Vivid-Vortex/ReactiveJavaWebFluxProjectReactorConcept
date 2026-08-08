# Thread Pools

## In Simple Terms

A thread pool is a group of ready-to-go worker threads waiting to pick up
tasks — so you're not paying the cost of spinning up a brand-new thread
every single time you need one, the same way a taxi rank keeps cabs ready
instead of building one from scratch for every ride. Reactor's `Schedulers`
are each backed by a different kind of thread pool, tuned for a different
kind of job.

## Simple Example

Comparing pool characteristics:

```java
// A fixed pool sized to CPU cores - for CPU-bound work
Scheduler cpuPool = Schedulers.parallel();

// A growable pool, capped at a high limit - for blocking/I/O work
Scheduler ioPool = Schedulers.boundedElastic();

// A pool with exactly one thread - for sequential work
Scheduler singlePool = Schedulers.single();

// A custom pool you configure yourself
Scheduler customPool = Schedulers.newBoundedElastic(
    50,          // max threads
    1000,        // max queued tasks per thread
    "my-custom-pool"
);
```

## Why It Matters

Picking the right pool size and type is a genuine performance concern:
- Too few threads for I/O-heavy work → requests pile up and things get slow.
- Too many threads for CPU-heavy work → threads spend more time switching
  back and forth than doing actual work, since there aren't more real cores
  to use.

Reactor's default pools are a sensible starting point, but real production
systems often tune custom pool sizes (like `Schedulers.newBoundedElastic(...)`)
based on what actually happens under real load, not guesswork.
