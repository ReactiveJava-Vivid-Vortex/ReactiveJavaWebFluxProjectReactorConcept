# Thread Affinity

## In Simple Terms

"Thread affinity" is just whether a piece of work is guaranteed to keep
running on the *same* thread the whole way through, or whether it might hop
between different threads at different points. In reactive pipelines, don't
count on staying on one thread — code before and after something
asynchronous (like a database call) can easily end up running on completely
different threads.

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

**Watch out for this:** anything relying on thread-local state (like some
logging context variables, or plain old `ThreadLocal` values) can quietly
"vanish" across these switches, because the new thread never had those
values to begin with. Reactor's own `Context` mechanism exists specifically
to carry that kind of data safely across thread switches, since
`ThreadLocal` just doesn't hold up in reactive code.

## Why It Matters

Knowing that reactive pipelines don't keep you on the same thread explains
why naive use of `ThreadLocal` (say, for tracking request info in logs)
often silently breaks in reactive code — and why Reactor's `Context` API
exists as the safe replacement.
