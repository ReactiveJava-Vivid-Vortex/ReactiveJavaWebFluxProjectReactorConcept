# request(n)

## In Simple Terms

`request(n)` is the core mechanism of backpressure — a subscriber calls it on its
`Subscription` to tell the publisher exactly how many items it's ready to receive
next. The publisher is contractually forbidden from sending more than the total
amount requested.

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

Without any explicit `request()` call, most convenience methods like
`.subscribe(consumer)` automatically request `Long.MAX_VALUE` (effectively
unbounded) on your behalf.

## Why It Matters

`request(n)` is the mechanism that prevents a fast producer from overwhelming a slow
consumer — it's the foundation that everything else in backpressure (strategies,
rate limiting, buffering) builds on top of.
