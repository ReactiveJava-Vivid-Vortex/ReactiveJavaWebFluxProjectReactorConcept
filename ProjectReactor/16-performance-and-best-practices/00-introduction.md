# Q1. Does Reactive Code Automatically Perform Well?

## Simple Explanation (Think of a Sports Car with the Handbrake On)

Reactive code that compiles and "looks reactive" isn't automatically fast. It's
like a sports car with the handbrake secretly on — looks right, but one hidden
mistake (a blocking call) ruins everything, and you might not notice until you hit
real traffic.

```
Looks reactive:  Mono<User> getUser() { return repo.findById(id); }
Actually reactive: ONLY if repo.findById() never blocks anywhere inside it
```

This topic is the final checklist for making sure your reactive code is
*genuinely* fast, not just *syntactically* reactive.

---

## Q2. What Is the #1 Rule, Repeated One More Time?

**Never block a thread meant to be non-blocking.**

```java
// BAD — blocks a precious event-loop thread
Mono.fromCallable(() -> { Thread.sleep(1000); return "done"; });

// GOOD — isolates the unavoidable blocking call
Mono.fromCallable(() -> { Thread.sleep(1000); return "done"; })
    .subscribeOn(Schedulers.boundedElastic());

// BEST — avoid blocking entirely with a genuinely non-blocking API
Mono.delay(Duration.ofSeconds(1)).map(t -> "done");
```

A single hidden blocking call (legacy JDBC, `Thread.sleep()`, a blocking HTTP
client) sitting on an event-loop/`parallel()` thread can silently stall **every
other request** sharing that thread — invisible in low-traffic testing,
catastrophic under real load.

---

## Q3. Is Reactive Memory Efficiency Automatic?

**No.** Certain common operators still accumulate unbounded state if misused on
huge or infinite streams:

```java
// Loads EVERYTHING into memory before emitting anything
hugeFlux.collectList().subscribe(list -> process(list));

// Remembers EVERY unique value seen — grows unboundedly with high-cardinality data
hugeFlux.distinct().subscribe();

// Can grow without limit if the consumer never catches up
fastProducerFlux.onBackpressureBuffer().subscribe(slowConsumer());
```

Fix: bound them explicitly.

```java
hugeFlux.buffer(1000).flatMap(batch -> processBatch(batch)).subscribe();

fastProducerFlux
    .onBackpressureBuffer(10_000, dropped -> log.warn("Dropped: {}", dropped))
    .subscribe(slowConsumer());
```

---

## Q4. Why Does Reactive Actually Help Throughput?

```
Blocking (thread-per-request):
  200-thread pool, each request waits 50ms on I/O
  -> throughput plateaus once all 200 threads are occupied waiting

Reactive (event-loop):
  8-16 threads NEVER blocked
  -> can service far more concurrent in-flight requests with the SAME thread count
```

The benefit is biggest for **I/O-bound, high-concurrency** workloads — it does
nothing for CPU-bound work, where the same computation takes the same CPU time
regardless of programming style.

---

## Q5. How Do I Pick the Right Operator for Performance?

```java
// Unnecessary overhead: flatMap() is for ASYNC transformations
flux.flatMap(item -> Mono.just(transform(item)));
// Correct and more efficient: map() for synchronous transformations
flux.map(item -> transform(item));

// Potentially unbounded concurrent calls — risky against a rate-limited API
flux.flatMap(item -> callExternalApi(item));
// Bounded concurrency — much safer
flux.flatMap(item -> callExternalApi(item), 10); // max 10 concurrent calls

// concatMap() when strict ordering matters (at the cost of concurrency)
flux.concatMap(item -> processInOrder(item));
```

---

## Q6. Interview-Style Q&A

### Why might a WebFlux app be slower than expected even though it "looks reactive"?

Almost always a hidden blocking call on an event-loop thread — audit every
external dependency (JDBC, HTTP clients, file I/O) for blocking behavior.

### Does `flatMap()` run its inner Monos in order?

**Not necessarily** — `flatMap()` runs them concurrently by default (up to a
configurable concurrency limit); use `concatMap()` if strict order matters.

### Is `.distinct()` safe to use on a huge Flux?

Only if the number of *unique* values is bounded — it keeps every unique value
seen in memory to detect future duplicates, so high-cardinality streams can grow
memory usage unboundedly.

---

## Q7. Summary — The Two-Question Audit

```
Is there ANY blocking call in this pipeline?
   │
   ├── YES ──▶ isolate on Schedulers.boundedElastic() (or replace with a non-blocking API)
   │
   └── NO — pipeline is genuinely non-blocking
              │
              ▼
        Are you accidentally buffering unbounded data?
        (collectList/distinct/unbounded onBackpressureBuffer on a huge/infinite stream)
              │
              ├── YES ──▶ bound it: buffer(n), limitRate(), bounded onBackpressureBuffer(n)
              └── NO  ──▶ you're following reactive best practices
```

### One sentence to remember

> **"Ask two questions of every pipeline you write: 'is anything secretly
> blocking?' and 'is anything secretly unbounded in memory?' — that's 90% of
> reactive performance work."**
