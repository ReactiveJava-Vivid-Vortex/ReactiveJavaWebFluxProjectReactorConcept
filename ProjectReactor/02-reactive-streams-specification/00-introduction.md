# Q1. What Is the Reactive Streams Specification?

## Simple Explanation (Think of a Broadcaster and a Viewer with a Remote)

Before Project Reactor existed, engineers from Netflix, Lightbend, Pivotal and
others agreed on a common **rulebook** so different async libraries could talk to
each other. That rulebook is the **Reactive Streams specification** — just four
tiny interfaces.

```
Publisher  (broadcaster — has data to send over time)
    ↓
Subscriber (viewer — wants to receive it)
```

But unlike a real TV broadcast, the viewer isn't forced to watch faster than they
can process. The viewer holds a **remote control** (`Subscription`):

```
Viewer: "Send me exactly 5 more episodes"   -> subscription.request(5)
Viewer: "Stop, I'm done"                     -> subscription.cancel()
```

This consumer-controlled pacing is called **backpressure** — the single most
important idea in the whole spec. It's what stops a fast producer from drowning a
slow consumer in more data than it can handle.

---

## Q2. What Are the Four Interfaces, Exactly?

| Interface | Role | One-Line Job |
|---|---|---|
| `Publisher<T>` | Broadcaster | Produces items; does nothing until `subscribe()` is called |
| `Subscriber<T>` | Viewer | Consumes items; reacts to `onSubscribe/onNext/onError/onComplete` |
| `Subscription` | Remote control | Lets the subscriber `request(n)` more items or `cancel()` |
| `Processor<T,R>` | Both at once | A `Subscriber` AND a `Publisher` — a bridge in the middle of a pipeline |

```java
public interface Publisher<T> { void subscribe(Subscriber<? super T> s); }
public interface Subscriber<T> {
    void onSubscribe(Subscription s);
    void onNext(T t);
    void onError(Throwable t);
    void onComplete();
}
public interface Subscription { void request(long n); void cancel(); }
```

`Processor` is rarely implemented directly anymore — modern Reactor code uses
`Sinks` instead (a later topic), which does the same "bridge" job more safely.

---

## Q3. What Does the Full Lifecycle Look Like?

```
1. subscriber.subscribe(publisher)
2. publisher hands back a Subscription  -> onSubscribe(subscription)
3. subscriber calls subscription.request(n)      (demand flows UPSTREAM)
4. publisher sends onNext(item) up to n times     (data flows DOWNSTREAM)
5. publisher sends exactly ONE terminal signal:
       onComplete()   — success
   OR  onError(t)     — failure
   (never both, nothing after)
```

**This exact grammar is the entire "vocabulary" of reactive streams:**
`onSubscribe onNext* (onError | onComplete)?` — see the dedicated
[[the-three-signal-types]] file below for a full deep-dive, since it's important
enough to deserve its own page.

---

## Q4. What Is `request(n)`, and Why Does It Matter So Much?

`request(n)` is how a `Subscriber` tells its `Subscription`, **"I am ready for `n`
more items."** The publisher is contractually forbidden from sending more than the
total requested.

```java
subscription.request(3);   // "send me exactly 3, then wait"
// publisher sends onNext() 3 times, then MUST STOP until asked again

subscription.request(Long.MAX_VALUE); // "send everything, no backpressure"
```

Without this rule, a fast publisher (reading a huge file) could flood a slow
subscriber (writing to a rate-limited API) with unbounded data — eventually an
`OutOfMemoryError`. `request(n)` is the mechanism that prevents that.

---

## Q5. What Is `cancel()`?

The subscriber's way of saying **"stop sending me data, I'm no longer
interested."** After `cancel()`, the publisher should stop emitting and release
resources.

```java
Disposable subscription = Flux.interval(Duration.ofSeconds(1))
    .subscribe(tick -> System.out.println("Tick: " + tick));

subscription.dispose(); // internally calls cancel() on the Subscription
```

In Spring WebFlux, if a browser closes its tab mid-stream, the framework calls
`cancel()` for you automatically — no wasted server work for an absent client.

---

## Q6. What Is "Demand Management"?

The running balance of "items requested but not yet delivered."

```
Subscriber calls request(5)   -> demand = 5
Publisher sends onNext() x3   -> demand = 2
Subscriber calls request(2)   -> demand = 4
Publisher sends onNext() x4   -> demand = 0 (must stop until asked again)
```

Good demand management prevents both **overproduction** (unbounded buffering) and
**underutilization** (consumer starving because it never asked for more).

---

## Q7. What Do `onNext`, `onComplete`, and `onError` Each Actually Mean?

| Signal | Meaning | Can Repeat? |
|---|---|---|
| `onNext(item)` | "Here's a new item" | Yes, 0 or more times |
| `onComplete()` | "Finished — everything succeeded" | No — at most once, terminal |
| `onError(t)` | "Something went wrong" | No — at most once, terminal |

```java
Flux.just(1, 2, 3)
    .subscribe(
        item -> System.out.println("onNext: " + item),
        error -> System.out.println("onError: " + error),
        () -> System.out.println("onComplete!")
    );
// onNext: 1 / onNext: 2 / onNext: 3 / onComplete!
```

`onComplete()` and `onError()` are **mutually exclusive** — a stream ends in
success *or* failure, never both, and nothing is signaled after either one.

---

## Q8. Interview-Style Q&A

### Can a Publisher send more items than requested?

**No.** That would violate the spec — a compliant publisher must never exceed
outstanding demand.

### Can `onNext` fire after `onComplete`?

**No.** Once a terminal signal (`onComplete`/`onError`) fires, the stream is over —
by definition, nothing more is ever signaled.

### Is `Processor` still commonly used in modern Reactor code?

**Rarely.** `Sinks` (a later topic) replaced most hand-rolled `Processor` usage —
it's safer and easier to use correctly.

### What's the difference between `Mono` and this raw specification?

`Mono`/`Flux` (Project Reactor's types) are production-ready **implementations**
of exactly this `Publisher` contract — the spec is the rulebook, Reactor is the
toolkit built on top of it.

---

## Q9. Summary

| Concept | Key Takeaway |
|---|---|
| Publisher | Produces data; inert until subscribed |
| Subscriber | Consumes data via 4 callback methods |
| Subscription | The demand/cancellation remote control |
| Processor | Subscriber + Publisher bridge (mostly replaced by Sinks) |
| Backpressure | Consumer controls producer's pace via `request(n)` |
| Signal grammar | `onSubscribe onNext* (onError \| onComplete)?` — the whole vocabulary |

### One sentence to remember

> **"A Publisher never sends more than a Subscriber asked for, and every stream
> ends in exactly one of two ways — success or failure, never both, never
> neither (unless it's still running)."**
