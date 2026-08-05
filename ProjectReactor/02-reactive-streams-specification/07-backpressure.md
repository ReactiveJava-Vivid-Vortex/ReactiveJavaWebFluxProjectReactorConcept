# Backpressure

## In Simple Terms

**Backpressure** is a mechanism that lets a slow **consumer** control how fast a
**producer** sends it data, so the consumer is never overwhelmed. Instead of the
producer blasting data as fast as possible, the consumer says "send me N items," and
the producer respects that limit.

Without backpressure, a fast publisher (e.g., reading a huge file) could push millions
of items into a slow consumer (e.g., writing to a rate-limited API), causing an
unbounded buildup of unprocessed items in memory — eventually crashing the app.

## Simple Example

Imagine a water tap (publisher) and a cup (subscriber):

```
No backpressure:  Tap fully open -> cup overflows onto the floor (data loss / crash)
With backpressure: Cup says "pour slowly, I'll tell you when I'm ready for more"
                   -> Tap only pours as much as requested, never overflows
```

In code:

```java
Flux.range(1, 1_000_000)
    .log()
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10); // only ever ask for 10 at a time
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println("Processing: " + value);
            request(1); // ask for 1 more once this one is done
        }
    });
```

## Why It Matters

Backpressure is the defining feature that separates Reactive Streams from a simple
callback-based async API. It's what lets a reactive pipeline safely connect a very
fast data source to a much slower consumer (like a network call or a database write)
without ever running out of memory.
