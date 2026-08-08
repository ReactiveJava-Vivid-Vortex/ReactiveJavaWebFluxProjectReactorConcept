# CPU-Bound Work

## In Simple Terms

CPU-bound work is anything that keeps the processor genuinely busy the
whole time — heavy math, encryption, hashing, resizing images, sorting big
piles of data. There's no waiting involved here — unlike I/O, the CPU is
actually grinding away nonstop.

## Simple Example

```java
Flux.range(1, 8)
    .parallel()                       // split across CPU-bound "rails"
    .runOn(Schedulers.parallel())     // use the CPU-optimized scheduler
    .map(n -> computeExpensiveHash(n)) // real CPU work
    .sequential()
    .subscribe(hash -> System.out.println("Hash: " + hash));
```

## CPU-Bound vs I/O-Bound

| Aspect                | CPU-Bound                       | I/O-Bound                          |
|------------------------|----------------------------------|--------------------------------------|
| Bottleneck             | Processor speed / core count     | Network/disk latency                |
| Scaling strategy       | More CPU cores help              | More concurrent connections help    |
| Right scheduler        | `Schedulers.parallel()`          | `Schedulers.boundedElastic()`       |
| Thread count needed    | ~= number of CPU cores            | Can be much higher than core count  |

## Why It Matters

Reactive programming's biggest superpower — not wasting threads while
they're waiting on something — doesn't really apply here. A CPU-heavy task
takes the same amount of processor time whether you write it reactively or
the plain old way. For CPU-bound work, the goal instead is just spreading
it correctly across the cores you actually have, using
`Schedulers.parallel()`, without piling on more threads than there are
cores to run them.
