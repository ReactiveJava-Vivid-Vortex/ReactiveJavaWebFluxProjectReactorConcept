# Q1. Does Reactive Code Automatically Run on Multiple Threads?

## Simple Explanation (Think of a Relay Race with One Runner)

Beginners assume "reactive" means "automatically parallel." **It does not.** By
default, a `Mono`/`Flux` pipeline runs entirely on whichever thread called
`.subscribe()` — like a relay race with only one runner who does every leg
themselves, never handing off the baton unless you explicitly tell them to.

```java
Mono.just("Hello")
    .map(v -> {
        System.out.println("map() runs on: " + Thread.currentThread().getName());
        return v.toUpperCase();
    })
    .subscribe(v -> System.out.println("subscribe() runs on: " + Thread.currentThread().getName()));

// Both print "main" — NOTHING switched threads!
```

---

## Q2. What Is a Scheduler?

An abstraction over a **pool of threads**, tuned for a specific kind of work. You
pick one, and Reactor runs your code on it.

| Scheduler | Pool Type | Use For |
|---|---|---|
| `Schedulers.parallel()` | Small, fixed (CPU-core sized) | CPU-bound, non-blocking work |
| `Schedulers.boundedElastic()` | Large, growable | Unavoidable blocking/I/O work |
| `Schedulers.single()` | ONE thread | Sequential, ordered, low-frequency work |
| `Schedulers.immediate()` | No-op — same thread | Default placeholder, rarely used directly |

---

## Q3. `publishOn()` vs `subscribeOn()` — The #1 Confusing Pair

```java
Mono.fromCallable(() -> {
    System.out.println("Source runs on: " + Thread.currentThread().getName());
    return "data";
})
.subscribeOn(Schedulers.boundedElastic())   // affects the WHOLE chain, from the SOURCE
.map(v -> {
    System.out.println("map runs on: " + Thread.currentThread().getName());
    return v;
})
.subscribe();
// Both print "boundedElastic-1" — subscribeOn affects EVERYTHING
```

```java
Flux.range(1, 3)
    .doOnNext(n -> System.out.println("Before: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.boundedElastic())   // only affects what's AFTER this point
    .doOnNext(n -> System.out.println("After: " + Thread.currentThread().getName()))
    .subscribe();
// "Before" prints "main", "After" prints "boundedElastic-1"
```

| | `subscribeOn()` | `publishOn()` |
|---|---|---|
| Affects | The ENTIRE chain, from the source, regardless of placement | Only what's AFTER it in the chain |
| How many matter | Only the FIRST one has effect | Can be used multiple times to switch repeatedly |

---

## Q4. The One Rule That Matters More Than All the Others

**Never block a thread meant to be non-blocking** — an event-loop or
`Schedulers.parallel()` thread. This is the #1 real-world reactive performance bug.

```java
// BAD — freezes a precious event-loop/parallel thread
Mono.fromCallable(() -> legacyBlockingCall());

// GOOD — isolates the unavoidable blocking call on a dedicated pool
Mono.fromCallable(() -> legacyBlockingCall())
    .subscribeOn(Schedulers.boundedElastic());
```

A single blocking call left on the wrong scheduler can silently stall **every
other request** sharing that small thread pool — invisible in low-traffic testing,
catastrophic under real load.

---

## Q5. CPU-Bound vs I/O-Bound — Which Scheduler for Which?

| | CPU-Bound | I/O-Bound |
|---|---|---|
| Bottleneck | Processor speed / core count | Network/disk latency |
| Example | Hashing, sorting, image processing | DB call, HTTP call, file read |
| Right scheduler | `Schedulers.parallel()` | `Schedulers.boundedElastic()` (if blocking) or none (if truly non-blocking) |
| Thread count needed | ~= number of CPU cores | Can be much higher than core count |

---

## Q6. What Is "Thread Affinity," and Why Should I Avoid `ThreadLocal`?

Reactive pipelines do **not** guarantee the same thread runs the whole chain —
code before/after a thread switch can run on entirely different threads.

```java
Mono.just("start")
    .doOnNext(v -> System.out.println("Step 1: " + Thread.currentThread().getName()))
    .publishOn(Schedulers.boundedElastic())
    .doOnNext(v -> System.out.println("Step 2: " + Thread.currentThread().getName()))
    .subscribe();
// Step 1: main
// Step 2: boundedElastic-1   <- DIFFERENT thread!
```

`ThreadLocal` values set on one thread simply won't be visible after a switch —
Reactor's `Context` API exists specifically as the thread-affinity-safe
alternative.

---

## Q7. Interview-Style Q&A

### If I don't call `subscribeOn()`/`publishOn()`, what thread does my pipeline run on?

Whatever thread called `.subscribe()` — no automatic switching happens.

### Can I use `publishOn()` more than once in a chain?

**Yes** — each call switches execution for everything downstream of it, so you
can bounce between schedulers multiple times.

### Can I use `subscribeOn()` more than once, with different effects?

**No** — only the first `subscribeOn()` encountered in a chain has any effect.

### Should I ever put a blocking JDBC call on `Schedulers.parallel()`?

**Never.** `parallel()` has a small, fixed thread count meant for CPU work — a
blocking call there wastes a disproportionate share of your CPU parallelism.
Always use `boundedElastic()` for blocking calls.

---

## Q8. Summary

| Concept | Key Takeaway |
|---|---|
| Default thread model | No automatic switching — runs on the subscribing thread |
| boundedElastic() | Large pool for unavoidable blocking/I/O work |
| parallel() | Small, fixed pool for genuine CPU-bound, non-blocking work |
| publishOn() | Switches everything AFTER this point; can be used multiple times |
| subscribeOn() | Switches the WHOLE chain from the source; only the first one counts |
| Thread affinity | NOT guaranteed — avoid `ThreadLocal`, use Reactor `Context` instead |

### One sentence to remember

> **"Reactive is not automatically parallel — it's your job to say WHERE
> (which scheduler) and WHEN (publishOn vs subscribeOn) any thread switch
> happens, and to never let a blocking call land on the wrong one."**
