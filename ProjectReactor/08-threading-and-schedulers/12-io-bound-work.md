# I/O-Bound Work

## In Simple Terms

**I/O-bound** work spends most of its time **waiting** for something external — a
database response, an HTTP call to another service, reading a file from disk. The
CPU is mostly idle during this wait; the bottleneck is network/disk latency, not
processing power.

## Simple Example

```java
// Non-blocking I/O-bound call - no scheduler switch needed, WebClient is already non-blocking
webClient.get()
    .uri("/api/users/{id}", userId)
    .retrieve()
    .bodyToMono(User.class)
    .subscribe(user -> System.out.println("Got user: " + user));

// Blocking I/O-bound call (legacy) - MUST be isolated on boundedElastic()
Mono.fromCallable(() -> legacyBlockingHttpCall(userId))
    .subscribeOn(Schedulers.boundedElastic())
    .subscribe(user -> System.out.println("Got user: " + user));
```

## Why It Matters

This is precisely the category of work reactive programming and non-blocking I/O
were designed to optimize. A truly non-blocking I/O call (like Spring's
`WebClient`) doesn't need a dedicated thread while waiting — the thread is freed
immediately and reused for other work, with a callback resuming processing once data
arrives. Legacy *blocking* I/O calls, however, must be explicitly isolated (with
`subscribeOn(Schedulers.boundedElastic())`) so they don't stall the small number of
event-loop threads meant for non-blocking work.
