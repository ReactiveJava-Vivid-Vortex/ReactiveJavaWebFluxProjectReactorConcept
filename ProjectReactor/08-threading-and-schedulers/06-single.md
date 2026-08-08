# single()

## In Simple Terms

`Schedulers.single()` gives you exactly one thread, shared by everything
that uses it. It's handy when you want tasks to run one after another, in
order, or when you want to avoid the overhead of multiple threads for a
small, occasional job.

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

A practical use — a lightweight background task that shouldn't compete with
your main worker pools:

```java
Schedulers.single().schedule(() -> System.out.println("Periodic housekeeping task"));
```

## Why It Matters

`Schedulers.single()` is useful whenever you need a dedicated lane that runs
things one at a time — writing safely to a single log file without
overlapping writes, or running low-priority background maintenance without
eating into your main I/O or CPU pools.
