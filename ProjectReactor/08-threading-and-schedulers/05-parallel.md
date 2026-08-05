# parallel()

## In Simple Terms

`Schedulers.parallel()` provides a **fixed-size** thread pool, sized to match the
number of available CPU cores. It's designed for **CPU-bound, non-blocking** work —
computations that keep the CPU busy (like hashing, sorting, or number crunching) —
not for I/O or blocking calls.

## Simple Example

```java
Flux.range(1, 4)
    .publishOn(Schedulers.parallel())
    .map(n -> {
        System.out.println("Computing on: " + Thread.currentThread().getName());
        return heavyComputation(n); // CPU-intensive, non-blocking
    })
    .subscribe(result -> System.out.println("Result: " + result));
```

Combined with `.parallel()` (the Flux operator, not the Scheduler) and `.runOn()` for
true multi-core parallel processing:

```java
Flux.range(1, 1000)
    .parallel(4)                          // split into 4 "rails"
    .runOn(Schedulers.parallel())         // run each rail on the parallel scheduler
    .map(n -> heavyComputation(n))
    .sequential()
    .subscribe();
```

## Why It Matters

Using `Schedulers.parallel()` for CPU-heavy work lets you actually use multiple CPU
cores concurrently — something the default single-threaded execution model won't do
for you automatically. **Never** run blocking calls (I/O, `Thread.sleep()`, blocking
JDBC) on this scheduler — its small, fixed thread count means a single blocking call
can stall a disproportionate share of your CPU-bound workload.
