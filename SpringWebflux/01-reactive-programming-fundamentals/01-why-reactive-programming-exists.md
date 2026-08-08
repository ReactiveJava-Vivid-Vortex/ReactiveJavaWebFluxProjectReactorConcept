# Why Reactive Programming Exists

## In Simple Terms

Reactive programming exists to fix a very real, everyday problem:
traditional web servers waste threads while waiting on slow things
(databases, other services). Every blocking request ties up an entire
thread for its whole duration, even though that thread does absolutely
nothing useful while it waits — it just sits there, taking up memory,
being tracked by the OS.

## Simple Example

Traditional Spring MVC (blocking) — one thread per request, frozen while
waiting:

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable String id) {
    return userRepository.findById(id); // thread BLOCKS here until DB responds
}
```

If the database takes 100ms to answer and you've got 10,000 requests
coming in at once, you'd need up to 10,000 threads just to keep everyone
"waiting" at the same time — a huge amount of memory spent on threads doing
nothing.

Spring WebFlux (reactive) — the thread gets freed up right away instead of
waiting:

```java
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable String id) { // returns immediately, no thread blocked
    return userRepository.findById(id);
}
```

## Why It Matters

With WebFlux, those same 10,000 requests can be handled by a small, fixed
group of threads (often just 4-16), because no thread ever sits frozen
waiting — it gets freed the moment it would otherwise block, and reused for
whatever else needs doing. That's the whole reason reactive programming
exists: much better use of resources when you've got a lot of concurrent,
I/O-heavy work going on.
