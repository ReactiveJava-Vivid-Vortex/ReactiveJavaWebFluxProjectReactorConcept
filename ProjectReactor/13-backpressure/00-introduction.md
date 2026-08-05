# Backpressure — Topic Overview

## What Is This Topic About? (In Simple Terms)

You met backpressure briefly in the Reactive Streams Specification topic — here we
go deeper into the mechanics and the practical decisions you have to make when a
**fast producer** meets a **slow consumer**.

The core mechanism is simple: a subscriber calls `request(n)` to say "I'm ready for
`n` more items," and a well-behaved publisher **never** sends more than that. This
is the subscriber staying in the driver's seat, controlling its own pace instead of
being flooded.

```java
Flux.range(1, 1_000_000)
    .subscribe(new BaseSubscriber<Integer>() {
        protected void hookOnSubscribe(Subscription s) { request(10); } // ask for 10 at a time
        protected void hookOnNext(Integer v) {
            process(v);
            request(1); // ask for one more once we're ready
        }
    });
```

But what happens when there's genuinely no way for the consumer to keep up, and data
would otherwise pile up? That's where **overflow strategies** come in — you
explicitly choose what to do: `BUFFER` (queue it, risk memory growth), `DROP`
(discard new excess items), `LATEST` (keep only the newest, discard older excess),
or `ERROR` (fail loudly instead of losing data silently). Choosing the right one is
a real design decision tied to your data's meaning — losing a stale sensor reading
is fine; silently dropping a financial transaction is not.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **request(n)** | The core mechanism: subscriber declares exactly how many items it's ready for next. |
| 2 | **Consumer demand** | The running total of items requested but not yet delivered — the consumer's stated capacity. |
| 3 | **Producer speed** | How fast a source can generate data — backpressure exists to temper this against actual consumer capacity. |
| 4 | **Overflow handling** | What happens when demand is exceeded — choose a strategy: `BUFFER`, `DROP`, `LATEST`, or `ERROR`. |
| 5 | **Backpressure strategies** | Reactor operators (`onBackpressureBuffer/Drop/Latest/Error()`) applying overflow handling directly in a pipeline. |
| 6 | **Rate limiting** | Deliberately slowing flow (`.delayElements()`, `.limitRate()`) to respect external constraints like API quotas. |

## How It All Fits Together

```
Fast Producer ──▶ [ request(n) enforced here ] ──▶ Slow Consumer
                          │
              What if producer wants to exceed demand?
                          │
        ┌─────────────────┼─────────────────┬───────────────┐
        ▼                 ▼                 ▼               ▼
    BUFFER            DROP              LATEST           ERROR
  (queue it,      (discard new       (keep newest,    (fail loudly
   risk memory)     excess items)   drop old excess)   instead of losing data)
```

The one-sentence summary of this whole topic: **backpressure is what lets a
firehose safely connect to a teacup** — without it, either the teacup overflows
(`OutOfMemoryError`) or data silently vanishes with no way to control which.
