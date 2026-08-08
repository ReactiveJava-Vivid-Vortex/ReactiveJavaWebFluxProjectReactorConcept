# Q1. What Is Backpressure?

## Simple Explanation (Think of a Firehose and a Teacup)

Imagine connecting a **firehose** (fast producer) directly to a **teacup** (slow
consumer), full blast, no control. The teacup overflows instantly — water (data)
goes everywhere, wasted, unmanaged.

```
No backpressure:  Firehose fully open -> teacup overflows onto the floor
With backpressure: Teacup says "pour slowly, tell me when I'm ready for more"
                    -> firehose only pours what's requested, never overflows
```

Backpressure is the **teacup's ability to control the firehose's flow rate** — the
consumer, not the producer, decides the pace.

---

## Q2. How Does `request(n)` Actually Implement This?

```java
Flux.range(1, 1_000_000)
    .subscribe(new BaseSubscriber<Integer>() {
        protected void hookOnSubscribe(Subscription s) { request(10); } // ask for 10 at a time
        protected void hookOnNext(Integer v) {
            process(v);
            request(1); // ask for one more once ready
        }
    });
```

The publisher is **contractually forbidden** from sending more than the
outstanding requested amount. This is the entire mechanism — nothing more magical
than "ask, then receive, then ask again."

---

## Q3. What If the Producer REALLY Can't Slow Down?

That's where **overflow strategies** come in — you explicitly choose what happens
when demand is exceeded:

| Strategy | Behavior | Best For |
|---|---|---|
| `BUFFER` | Queue excess items in memory | Bursty traffic that should eventually process, not be lost |
| `DROP` | Discard new items that exceed demand | High-frequency data where losing some is fine |
| `LATEST` | Keep only the newest, discard older excess | Live state where only the newest value matters |
| `ERROR` | Fail loudly instead of losing data silently | When silent data loss is unacceptable |

```java
Flux.range(1, 1000)
    .onBackpressureDrop(dropped -> System.out.println("Dropped: " + dropped))
    .subscribe(slowConsumer());

Flux.range(1, 1000)
    .onBackpressureLatest()
    .subscribe(slowConsumer());
```

**This is a real design decision**, not a technical afterthought: losing a stale
sensor reading is fine; silently dropping a financial transaction is not.

---

## Q4. What Is "Rate Limiting," and How Is It Different from Backpressure?

Backpressure is about **technical inability to keep up**. Rate limiting is about
**deliberately** slowing down to respect an **external** constraint — like a
third-party API's requests-per-second quota.

```java
Flux.range(1, 10)
    .delayElements(Duration.ofMillis(200)) // emit at most one every 200ms
    .subscribe(n -> System.out.println("Calling API with: " + n));

Flux.fromIterable(userIds)
    .flatMap(id -> callExternalApi(id), 5) // max concurrency of 5, at any time
    .subscribe(response -> System.out.println("Got: " + response));
```

---

## Q5. Interview-Style Q&A

### Can a publisher legally send more items than requested?

**No.** Doing so violates the Reactive Streams contract.

### If I never call `request()` explicitly, does backpressure still apply?

Most convenience methods (like `.subscribe(consumer)`) automatically request
`Long.MAX_VALUE` (effectively unbounded) — so backpressure exists in theory, but
isn't actively limiting anything unless you request in controlled batches.

### What's the risk of choosing `BUFFER` without a bound?

Unbounded buffering can still lead to `OutOfMemoryError` if the producer stays
faster than the consumer indefinitely — always prefer a bounded buffer with an
overflow action when possible.

---

## Q6. Summary

```
Fast Producer ──▶ [ request(n) enforced here ] ──▶ Slow Consumer
                          │
              What if producer wants to exceed demand?
                          │
        ┌─────────────────┼─────────────────┬───────────────┐
        ▼                 ▼                 ▼               ▼
    BUFFER            DROP              LATEST           ERROR
  (queue, risk      (discard new      (keep newest,     (fail loudly
   memory growth)    excess items)    drop old excess)   instead of losing data)
```

### One sentence to remember

> **"Backpressure is what lets a firehose safely connect to a teacup — without
> it, either the teacup overflows or water silently vanishes with no way to
> control which."**
