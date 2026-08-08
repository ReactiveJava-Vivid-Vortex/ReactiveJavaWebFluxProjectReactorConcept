# Thread Switching

## In Simple Terms

"Thread switching" just means moving execution over to a different thread
at some point in the pipeline, using `publishOn()` or `subscribeOn()`. Once
you switch, everything *after* that point runs on the new thread, like a
relay race handing the baton to a new runner — until another switch happens
further down.

## Simple Example

```java
Mono.just("data")
    .doOnNext(v -> System.out.println("Before switch: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.boundedElastic())
    .doOnNext(v -> System.out.println("After switch: " + Thread.currentThread().getName()))
    .subscribe();
```

Output (thread names will differ from `main`):
```
Before switch: main
After switch: boundedElastic-1
```

Everything after `.publishOn(...)` now runs on a `boundedElastic` worker
thread instead of `main`.

## Why It Matters

Deliberately switching threads matters for a few reasons:
1. Getting a blocking call off a limited thread (moving it onto
   `boundedElastic()`).
2. Putting CPU-heavy work on a `parallel()` scheduler so it can use multiple
   cores.
3. Coming back to a specific thread afterward if you need to (like a UI
   thread in a desktop app).

Without understanding this, it's easy to accidentally leave blocking code
running on threads that were only ever meant for fast, non-blocking work.
