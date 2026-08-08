# Scheduler Best Practices

## In Simple Terms

Picking the right scheduler for the right job is one of the most important
skills for writing reactive code that actually performs well. Here are the
main rules of thumb worth keeping in your back pocket.

## The Golden Rules

1. **Never let a blocking call sit on an event-loop or `parallel()`
   thread.** If you have to call something blocking (old JDBC,
   `Thread.sleep()`, blocking file reads), wrap it and move it to
   `Schedulers.boundedElastic()` with `.subscribeOn()`.

   ```java
   Mono.fromCallable(() -> legacyBlockingCall())
       .subscribeOn(Schedulers.boundedElastic());
   ```

2. **Only use `Schedulers.parallel()` for genuinely CPU-heavy, non-blocking
   work.** Don't use it for I/O — its thread count is capped at your CPU
   count, so a single stuck thread there eats up a disproportionate chunk
   of your available parallelism.

3. **Prefer non-blocking tools (WebClient, R2DBC) over wrapping blocking
   ones.** `subscribeOn(boundedElastic())` is a decent escape hatch, but a
   truly non-blocking driver skips the overhead entirely.

4. **Don't build unbounded custom schedulers.** Size custom pools
   deliberately (`Schedulers.newBoundedElastic(...)`) based on real load
   testing, not a guess.

5. **Keep thread switches to a minimum.** Every `.publishOn()`/`.subscribeOn()`
   call has a small cost — don't sprinkle them in without a specific reason.

6. **Clean up custom schedulers when you're done with them**, if you built
   one yourself (e.g., `Scheduler myScheduler = Schedulers.newParallel(...)`),
   so you don't leak threads:

   ```java
   myScheduler.dispose();
   ```

## Why It Matters

Getting the scheduler choice wrong is one of the most common — and most
damaging — mistakes in production reactive systems. A single blocking call
accidentally left on an event-loop thread can quietly drag down an entire
app's performance, often only showing up once real traffic hits it hard.
