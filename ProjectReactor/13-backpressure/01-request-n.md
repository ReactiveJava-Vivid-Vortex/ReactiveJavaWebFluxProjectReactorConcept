# request(n)

## In Simple Terms

`request(n)` is how a subscriber tells the source "send me `n` more, and
not a single item beyond that." It's the whole mechanism backpressure is
built on — like telling a waiter "just bring two dishes for now," instead
of letting the kitchen pile up plates faster than you can eat.

## Simple Example

```java
Flux.range(1, 1000)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10); // only ask for 10 to start
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println("Processing: " + value);
            if (value % 10 == 0) {
                request(10); // ask for 10 more once we've used up the last batch
            }
        }
    });
```

If you never call `request()` yourself, most of the convenience methods
like `.subscribe(consumer)` quietly ask for everything (`Long.MAX_VALUE`)
on your behalf.

## Why It Matters

`request(n)` is what stops a fast producer from flooding a slow consumer —
it's the foundation everything else about backpressure (strategies, rate
limiting, buffering) is built on top of.
