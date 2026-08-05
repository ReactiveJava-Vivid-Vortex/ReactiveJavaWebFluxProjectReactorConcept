# Reactive Programming Fundamentals — Topic Overview

## What Is This Topic About? (In Simple Terms)

This topic is the "why" behind Spring WebFlux, before you write a single line of
WebFlux code. The core problem: traditional Spring MVC gives each incoming request
its own dedicated thread, which **freezes** for the entire duration of any slow I/O
(a database call, an external API call). Freeze enough threads at once (say, 10,000
concurrent slow requests) and you run out of memory/threads long before the CPU
itself is actually busy.

WebFlux flips this: it uses a small, fixed pool of non-blocking **event-loop**
threads that are never frozen — the instant a thread would have to wait, it's
released to service a different request, and resumes the original one later when
data arrives.

```java
// Spring MVC — thread FREEZES here until the DB responds
@GetMapping("/users/{id}")
public User getUser(@PathVariable String id) { return repo.findById(id); }

// Spring WebFlux — returns immediately, no thread ever blocked
@GetMapping("/users/{id}")
public Mono<User> getUser(@PathVariable String id) { return repo.findById(id); }
```

Underpinning all of this is the **Reactive Manifesto**: a well-built reactive system
should be **Responsive** (bounded, predictable response times), **Resilient**
(failures in one part don't cascade), **Elastic** (scales gracefully under load
spikes), and **Message Driven** (components talk via async events, which is what
enables the other three). Everything WebFlux does — `Mono`/`Flux`, `WebClient`,
R2DBC — is designed in service of these four traits.

Crucially, WebFlux isn't automatically the right choice everywhere — it shines for
high-concurrency, I/O-heavy workloads, but adds real complexity that isn't worth it
for CPU-bound work, low-concurrency apps, or teams reliant on blocking libraries
(like JPA) with no migration plan.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **Why Reactive Programming exists** | Blocking servers waste threads waiting on I/O; reactive frees threads instead of freezing them. |
| 2 | **Blocking vs Non-Blocking I/O** | Blocking freezes the thread until data arrives; non-blocking moves on and gets notified later. |
| 3 | **Reactive Manifesto** | The 4 traits of a good reactive system: Responsive, Resilient, Elastic, Message Driven. |
| 4 | **Responsive** | Timely, bounded response times, even under load or failure — fail fast and clearly, don't hang. |
| 5 | **Resilient** | Failures in one part (a downstream service) don't cascade into a total system outage. |
| 6 | **Elastic** | Stays responsive across a wide range of load, without needing proportionally more hardware. |
| 7 | **Message Driven** | Components communicate via async events, not direct blocking calls — the foundation enabling the other 3 traits. |
| 8 | **Publisher/Subscriber model** | Everything in WebFlux — requests, DB calls, HTTP calls — speaks the same Mono/Flux publisher language. |
| 9 | **Backpressure** | Lets a slow client control how fast a server streams data, preventing memory overload. |
| 10 | **Why WebFlux scales better than MVC** | Small event-loop pool never blocks vs. MVC's thread-per-request pool that freezes on I/O. |
| 11 | **When NOT to use WebFlux** | CPU-bound work, low concurrency, or heavy reliance on blocking libraries (JPA) with no migration plan. |
| 12 | **Browser streaming demo** | A `Flux.interval()` endpoint visibly streaming data to a browser incrementally — makes streaming concrete. |
| 13 | **Cancellation of requests** | WebFlux auto-cancels the pipeline if a client disconnects — no wasted work for an absent client. |
| 14 | **Reactive pipeline** | The entire request lifecycle — controller → service → repository/WebClient → response — as one continuous non-blocking chain. |

## How It All Fits Together

```
Client request arrives
        │
        ▼
Small pool of event-loop threads (never blocks)
        │
        ▼
Mono/Flux pipeline: controller → service → repository/WebClient
        │            (all non-blocking, Publisher/Subscriber under the hood)
        ▼
Response streamed back — backpressure paces it to the client's speed
        │
   (if client disconnects, cancellation propagates automatically)
```

Internalize this: WebFlux's value isn't "faster per-request" — it's "the same
hardware handles vastly more *concurrent* slow requests." Keep that distinction in
mind for every topic that follows.
