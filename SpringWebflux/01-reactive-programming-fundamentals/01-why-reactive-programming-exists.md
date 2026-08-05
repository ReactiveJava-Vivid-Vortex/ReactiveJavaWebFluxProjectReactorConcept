# Why Reactive Programming Exists

## In Simple Terms

Reactive programming exists to solve a specific, real problem: **traditional
blocking web servers waste threads while waiting on slow I/O** (databases, external
APIs). Each blocking request ties up a whole thread for its entire duration, even
though the thread does nothing useful during the wait — just holding memory and
being tracked by the OS scheduler.

## Simple Example

Traditional Spring MVC (blocking) — one thread per request, frozen during I/O wait:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable String id) {
    return userRepository.findById(id); // thread BLOCKS here until DB responds
}
```

If the database takes 100ms to respond and you have 10,000 concurrent requests, you'd
need up to 10,000 threads just to keep all requests "waiting" simultaneously — a huge
memory cost for a lot of doing-nothing.

Spring WebFlux (reactive) — the thread is released immediately during the wait:

```java
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable String id) {
    return userRepository.findById(id); // returns immediately, no thread blocked
}
```

## Why It Matters

With WebFlux, the same 10,000 concurrent requests can be handled by a small, fixed
pool of event-loop threads (often just 4-16), because no thread is ever frozen
waiting — it's freed the instant it would otherwise block, and reused for other
incoming work. This is the entire motivation behind reactive programming's existence:
dramatically better resource efficiency under high, I/O-heavy concurrency.
