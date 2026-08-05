# CPU-Bound Work

## In Simple Terms

**CPU-bound** work is any computation that keeps the processor busy the entire time
— math-heavy calculations, encryption/hashing, image resizing, sorting large
datasets. Unlike I/O, there's no "waiting" involved; the CPU is doing real work
continuously.

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

Reactive programming's main benefit (avoiding wasted threads while waiting on I/O)
does **not** apply to CPU-bound work — a CPU-heavy task takes the same amount of
processor time whether written reactively or synchronously. For CPU-bound work, the
goal instead is to correctly parallelize across available cores using
`Schedulers.parallel()`, without over-subscribing more threads than there are actual
cores to run them on.
