# Scheduler Best Practices

## In Simple Terms

Choosing the right scheduler for the right job is one of the most important skills in
writing correct, performant reactive code. Here are the key rules of thumb.

## The Golden Rules

1. **Never block on an event-loop / `parallel()` thread.** If you must call blocking
   code (legacy JDBC, `Thread.sleep()`, blocking file I/O), wrap it and move it to
   `Schedulers.boundedElastic()` using `.subscribeOn()`.

   ```java
   Mono.fromCallable(() -> legacyBlockingCall())
       .subscribeOn(Schedulers.boundedElastic());
   ```

2. **Use `Schedulers.parallel()` only for genuinely CPU-bound, non-blocking work.**
   Don't use it for I/O — its thread count is capped at CPU core count, and a single
   blocked thread there wastes a disproportionate share of your parallelism.

3. **Prefer non-blocking APIs (WebClient, R2DBC) over wrapping blocking ones.**
   `subscribeOn(boundedElastic())` is a good escape hatch, but a truly non-blocking
   driver avoids the overhead entirely.

4. **Don't create unbounded custom schedulers.** Size custom thread pools
   deliberately (`Schedulers.newBoundedElastic(...)`) based on real load testing, not
   guesswork.

5. **Minimize the number of thread switches.** Every `.publishOn()`/`.subscribeOn()`
   call has a small overhead; don't add them without a specific reason.

6. **Dispose of custom schedulers when done**, if you created them manually (e.g.,
   `Scheduler myScheduler = Schedulers.newParallel(...)`), to avoid leaking threads:

   ```java
   myScheduler.dispose();
   ```

## Why It Matters

Getting scheduler choice wrong is one of the most common and damaging mistakes in
production reactive systems — a single blocking call accidentally left on an
event-loop thread can silently degrade the performance of an entire application under
load, often only showing up once traffic increases significantly.
