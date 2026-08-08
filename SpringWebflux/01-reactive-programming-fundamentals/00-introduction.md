# Q1. What Problem Does Spring WebFlux Actually Solve?

## Simple Explanation (Think of a Restaurant with One Waiter per Table vs a Shared Team)

Traditional Spring MVC assigns **one dedicated waiter (thread) per table
(request)**. If that table's order takes forever to cook (a slow DB/API call), the
waiter just **stands there frozen**, unable to serve anyone else, for the entire
wait.

```
Spring MVC:  Table A gets Waiter A, frozen until food's ready
             Table B gets Waiter B, frozen until food's ready
             ... 10,000 tables need 10,000 waiters (threads)!
```

Spring WebFlux uses a **small shared team** of waiters who never freeze — the
instant a wait begins, that waiter moves to help another table, and comes back the
moment the food is actually ready.

```
Spring WebFlux:  8-16 waiters, NEVER frozen
                 Serve 10,000 tables by constantly cycling to whoever's food is ready
```

```java
// Spring MVC — thread FREEZES here until the DB responds
@GetMapping("/users/{id}")
public User getUser(@PathVariable String id) { return repo.findById(id); }

// Spring WebFlux — returns immediately, no thread ever blocked
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable String id) { return repo.findById(id); }
```

---

## Q2. What Is the Reactive Manifesto, and Why Should I Care?

Four traits a well-built reactive system should have:

| Trait | Meaning | WebFlux Feature Enabling It |
|---|---|---|
| **Responsive** | Timely, bounded response times, even under load | `.timeout()` |
| **Resilient** | Failures in one part don't cascade | `.onErrorResume()`, fallbacks |
| **Elastic** | Stays responsive across varying load | Event-loop thread model |
| **Message Driven** | Components talk via async events, not blocking calls | `Mono`/`Flux` everywhere |

```
        Responsive
            ^
            |
Resilient <-+-> Elastic
            |
     Message Driven (the foundation enabling the other 3)
```

---

## Q3. Blocking vs Non-Blocking I/O — Applied to WebFlux Specifically

```java
// Blocking: thread FREEZES until data arrives — one thread per slow operation
int data = socket.getInputStream().read();

// Non-blocking: thread is freed instantly, notified LATER when data is ready
// This is what Netty (WebFlux's engine) does under the hood, automatically
```

WebFlux runs on **Netty**, using I/O multiplexing (`epoll`/`kqueue`) — this is
literally why a handful of threads can serve tens of thousands of concurrent
connections.

---

## Q4. Why Does WebFlux Scale Better Than Spring MVC? (The Concrete Numbers)

```
Spring MVC, 200-thread pool, 50ms downstream calls:
  200 concurrent slow requests -> ALL threads busy/blocked
  -> request #201 must WAIT in queue, even though the CPU is mostly idle

Spring WebFlux, ~8-16 event-loop threads:
  10,000 concurrent slow requests -> threads NEVER blocked waiting
  -> they cycle through servicing whichever request's data is ready
```

**This gap is biggest for I/O-bound, high-concurrency workloads** — exactly the
profile of most public APIs and microservices.

---

## Q5. When Should I NOT Use WebFlux?

| Situation | Verdict |
|---|---|
| Team unfamiliar with reactive debugging/testing | ❌ Real learning-curve cost may not be worth it |
| Heavy reliance on blocking libraries (JPA) with no migration plan | ❌ You'd wrap everything in `boundedElastic()`, losing most of the benefit |
| CPU-bound workload (image processing) | ❌ Reactive doesn't speed up CPU work |
| Low concurrency (internal admin tool, 10 users) | ❌ Added complexity isn't worth it |
| High-concurrency, I/O-heavy public API | ✅ WebFlux shines here |

---

## Q6. Everything Is a Publisher — What Does That Actually Mean?

```java
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable String id) { // Mono = Publisher of 0-1 User
        return userRepository.findById(id); // WebFlux subscribes internally
    }
}
```

You never call `.subscribe()` in a controller — WebFlux subscribes to whatever
`Mono`/`Flux` you return, streams the result back, and — critically — **the
entire vocabulary of that data flow is exactly three signal types**:
`onNext`/`onComplete`/`onError`. See [[the-three-signal-types]] in the Project
Reactor notes for the full breakdown of that grammar.

---

## Q7. What Happens If a Client Disconnects Mid-Request?

```java
@GetMapping(value = "/live-feed", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<String> liveFeed() {
    return Flux.interval(Duration.ofSeconds(1))
        .doOnCancel(() -> System.out.println("Client disconnected — stopping feed"))
        .map(tick -> "Update #" + tick);
}
```

WebFlux **automatically cancels** the underlying pipeline — the connection close
propagates as a `cancel()` signal down through the entire chain, so no server work
continues for a client that's no longer listening.

---

## Q8. Interview-Style Q&A

### Does WebFlux make a single request faster?

**No.** The database/network latency is identical either way. WebFlux improves
**throughput under high concurrency**, not per-request speed.

### Is a WebFlux controller method's Mono/Flux subscribed to immediately when the method is called?

**No.** WebFlux subscribes to it as part of handling the HTTP request lifecycle —
the method just *returns a description* of what should happen.

### Can you mix blocking and non-blocking code in a WebFlux app?

**Yes**, but any unavoidable blocking call must be isolated on
`Schedulers.boundedElastic()` — never left on an event-loop thread.

---

## Q9. Summary

```
Client request arrives
        │
        ▼
Small pool of event-loop threads (never blocks)
        │
        ▼
Mono/Flux pipeline: controller → service → repository/WebClient
        │            (all non-blocking, built from onNext/onComplete/onError signals)
        ▼
Response streamed back — backpressure paces it to the client's speed
        │
   (if client disconnects, cancellation propagates automatically)
```

### One sentence to remember

> **"WebFlux's value isn't 'faster per-request' — it's 'the same hardware
> handles vastly more concurrent slow requests,' by never letting a thread
> freeze while waiting."**
