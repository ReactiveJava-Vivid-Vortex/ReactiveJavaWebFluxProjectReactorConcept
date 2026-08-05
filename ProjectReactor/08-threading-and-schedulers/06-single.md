# single()

## In Simple Terms

`Schedulers.single()` provides **one single, reusable thread** shared across all
tasks submitted to it. It's useful when you need sequential, ordered execution of
tasks, or when you specifically want to avoid the overhead of multiple threads for a
lightweight, low-frequency workload.

## Simple Example

```java
Flux.just("Task A", "Task B", "Task C")
    .publishOn(Schedulers.single())
    .subscribe(task -> System.out.println(task + " on " + Thread.currentThread().getName()));
```

Output — notice all tasks run on the *same* single thread:
```
Task A on single-1
Task B on single-1
Task C on single-1
```

A practical use — a lightweight background task scheduler that shouldn't compete
with your main worker pools:

```java
Schedulers.single().schedule(() -> System.out.println("Periodic housekeeping task"));
```

## Why It Matters

`Schedulers.single()` is handy when you specifically need a dedicated, sequential
execution context — e.g., writing to a single log file safely without concurrent
writes, or running low-priority background maintenance tasks without consuming
threads from your main I/O or CPU pools.
