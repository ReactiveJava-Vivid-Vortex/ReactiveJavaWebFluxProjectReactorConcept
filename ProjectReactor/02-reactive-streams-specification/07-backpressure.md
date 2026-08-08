# Backpressure

## In Simple Terms

Backpressure just means: **the consumer decides how fast data flows, not the
producer.** Instead of a fast source blasting out data as quickly as it can, the
consumer says "send me N items," and the source respects that.

Without it, a fast source (say, reading a huge file) could dump millions of items
onto a slow consumer (say, something writing to a rate-limited API) faster than it
can keep up — and all those unprocessed items would just pile up in memory until
the app crashes.

## Simple Example

Picture a water tap (the publisher) filling a cup (the subscriber):

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

Backpressure is what really sets Reactive Streams apart from a plain
callback-based async API. It's the thing that lets you safely hook up a very fast
data source to a much slower consumer — like a network call or a database write —
without ever running out of memory.
