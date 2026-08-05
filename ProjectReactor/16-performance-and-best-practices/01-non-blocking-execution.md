# Non-Blocking Execution

## In Simple Terms

The single most important performance rule in reactive programming: **never block a
thread that's meant to be non-blocking** (event-loop threads, `Schedulers.parallel()`
threads). Blocking even briefly on one of these threads can stall many unrelated
concurrent requests sharing that same small thread pool.

## Simple Example

```java
// BAD: blocks a precious event-loop thread
@GetMapping("/bad")
public Mono<String> badExample() {
    return Mono.fromCallable(() -> {
        Thread.sleep(1000); // BLOCKS whatever thread runs this!
        return "done";
    });
}

// GOOD: isolates the blocking call on a dedicated pool
@GetMapping("/good")
public Mono<String> goodExample() {
    return Mono.fromCallable(() -> {
        Thread.sleep(1000); // still blocks, but on an isolated thread
        return "done";
    }).subscribeOn(Schedulers.boundedElastic());
}

// BEST: avoid blocking entirely, use a non-blocking API
@GetMapping("/best")
public Mono<String> bestExample() {
    return Mono.delay(Duration.ofSeconds(1)).map(t -> "done"); // fully non-blocking
}
```

## Why It Matters

A single accidental blocking call on a shared event-loop thread pool (which might
have as few as 4-8 threads for the entire application) can silently degrade
performance for **every** concurrent request being handled by that pool — not just
the one that made the blocking call. This is the #1 real-world reactive performance
bug.
