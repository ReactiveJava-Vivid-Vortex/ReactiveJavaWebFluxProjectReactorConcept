# Scheduler Concept

## In Simple Terms

A `Scheduler` is basically a pool of worker threads that reactive operators
can borrow when they need somewhere to run. Instead of managing raw
threads yourself, you just pick a `Scheduler` that suits the job — waiting
on I/O, crunching numbers, whatever — and Reactor takes care of handing
work off to it.

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

Different items may land on different threads from the pool, depending on
how things get scheduled.

## Why It Matters

Schedulers let you deliberately choose *where* work happens, matching each
kind of task to the right kind of pool — a small, fixed pool for CPU work
(`parallel()`), a bigger, flexible pool for blocking work
(`boundedElastic()`), or a single dedicated thread (`single()`). Picking the
wrong one for the job is one of the most common mistakes people make with
reactive code.
