# Non-Blocking Execution

## In Simple Terms

Here's the single most important performance rule in reactive programming:
never let something blocking sit on a thread that's supposed to stay free —
event-loop threads, `Schedulers.parallel()` threads. Even a brief block on
one of these can stall a whole bunch of unrelated requests that were
sharing that same small pool of threads.

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

A single accidental blocking call sitting on a shared event-loop pool
(which might only have 4-8 threads for the whole app) can quietly slow down
*every* concurrent request going through that pool — not just the one that
caused it. This is, in practice, the number one reactive performance bug
people run into.
