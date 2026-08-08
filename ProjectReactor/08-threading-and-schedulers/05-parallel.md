# parallel()

## In Simple Terms

`Schedulers.parallel()` gives you a small, fixed set of threads — one per
CPU core, roughly. It's built for work that keeps the CPU genuinely busy,
like number crunching or hashing, not for waiting on a network call or a
disk read.

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

Combined with the `.parallel()` operator (a different thing from the
`Scheduler` of the same name) and `.runOn()`, for true multi-core work:

```java
Flux.range(1, 1000)
    .parallel(4)                          // split into 4 "rails"
    .runOn(Schedulers.parallel())         // run each rail on the parallel scheduler
    .map(n -> heavyComputation(n))
    .sequential()
    .subscribe();
```

## Why It Matters

Using `Schedulers.parallel()` for CPU-heavy work lets you actually spread it
across multiple cores at once — something the default single-threaded
behavior won't give you for free. Never put a blocking call here — the
thread count is small and fixed, so a single stuck thread can hold back a
disproportionate chunk of your whole CPU-bound workload.
