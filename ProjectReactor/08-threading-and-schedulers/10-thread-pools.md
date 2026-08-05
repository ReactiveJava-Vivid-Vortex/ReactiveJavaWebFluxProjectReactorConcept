# Thread Pools

## In Simple Terms

A **thread pool** is a managed collection of reusable worker threads, ready to
execute submitted tasks, so you don't pay the cost of creating and destroying a new
thread for every single unit of work. Project Reactor's `Schedulers` are all backed
by different thread pool configurations, each tuned for a different kind of workload.

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

Choosing the right thread pool size and type is a real performance concern:
- Too few threads for I/O-bound work → requests queue up and latency spikes.
- Too many threads for CPU-bound work → excessive context switching hurts
  throughput, since there aren't more physical cores to use.

Reactor's default schedulers are sensible starting points, but production systems
often tune custom pool sizes (like `Schedulers.newBoundedElastic(...)`) based on
observed load and available hardware.
