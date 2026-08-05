# Scheduler Concept

## In Simple Terms

A `Scheduler` in Project Reactor is an abstraction over **a pool of threads** that
reactive operators can use to execute work. Instead of managing raw `Thread` or
`ExecutorService` objects yourself, you pick a `Scheduler` suited to your workload
(I/O-bound, CPU-bound, etc.), and Reactor handles submitting tasks to it.

```java
Scheduler scheduler = Schedulers.boundedElastic();

Mono.just("task")
    .publishOn(scheduler)
    .subscribe(value -> System.out.println("Running on: " + Thread.currentThread().getName()));
```

## Simple Example

```java
Flux.range(1, 5)
    .publishOn(Schedulers.parallel())
    .map(n -> {
        System.out.println("Processing " + n + " on " + Thread.currentThread().getName());
        return n * n;
    })
    .subscribe();
```

Each item may be processed by a different thread from the `parallel` scheduler's
pool, depending on scheduling.

## Why It Matters

Schedulers let you deliberately choose **where** work runs, matching the right kind
of thread pool to the right kind of task — a small, fixed pool for CPU work
(`parallel()`), a larger, growable pool for blocking work (`boundedElastic()`), or a
single dedicated thread (`single()`). Picking the wrong scheduler for a given
workload is one of the most common reactive performance mistakes.
