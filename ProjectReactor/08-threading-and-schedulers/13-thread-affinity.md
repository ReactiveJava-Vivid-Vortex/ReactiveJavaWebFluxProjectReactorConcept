# Thread Affinity

## In Simple Terms

**Thread affinity** refers to whether a piece of work is guaranteed (or expected) to
keep running on the *same* thread throughout its execution, versus potentially
hopping between different threads at different stages. In reactive pipelines, thread
affinity is generally **not guaranteed** across asynchronous boundaries — code before
and after an async operation (like a database call) may run on entirely different
threads.

## Simple Example

```java
Mono.just("start")
    .doOnNext(v -> System.out.println("Step 1 on: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.boundedElastic())
    .doOnNext(v -> System.out.println("Step 2 on: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.parallel())
    .doOnNext(v -> System.out.println("Step 3 on: " + Thread.currentThread().getName()))
    .subscribe();
```

Output (three different thread names, no affinity preserved):
```
Step 1 on: main
Step 2 on: boundedElastic-1
Step 3 on: parallel-1
```

**Important gotcha:** things that rely on thread-local state (like some logging MDC
contexts, or `ThreadLocal` variables) can silently "disappear" across these thread
switches, because the new thread doesn't have the same `ThreadLocal` values as the
old one. Reactor provides its own `Context` mechanism specifically to carry
contextual data safely across these thread switches, since `ThreadLocal` doesn't work
reliably in reactive pipelines.

## Why It Matters

Understanding that reactive pipelines don't preserve thread affinity by default
explains why naive use of `ThreadLocal` (e.g., for request-scoped logging context)
often breaks in reactive code — and why Reactor's `Context` API exists as the
correct, thread-affinity-safe alternative.
