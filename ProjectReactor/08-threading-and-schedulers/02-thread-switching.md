# Thread Switching

## In Simple Terms

"Thread switching" in a reactive pipeline means moving execution from one thread to
another at a specific point, using `publishOn()` or `subscribeOn()`. Once you switch,
every operator **downstream** of that point runs on the new thread, until another
switch happens.

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

Everything after `.publishOn(...)` in the chain now executes on a `boundedElastic`
worker thread instead of `main`.

## Why It Matters

Deliberate thread switching is essential for:
1. Moving a blocking call off of a limited event-loop thread (e.g., onto
   `boundedElastic()`).
2. Moving CPU-intensive work onto a `parallel()` scheduler to use multiple cores.
3. Returning execution to a specific thread afterward if needed (e.g., a UI thread in
   a desktop app).

Without understanding thread switching, it's easy to accidentally leave blocking code
running on threads meant only for fast, non-blocking work.
