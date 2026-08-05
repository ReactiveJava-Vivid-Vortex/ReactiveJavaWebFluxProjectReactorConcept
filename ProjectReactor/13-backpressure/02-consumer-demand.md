# Consumer Demand

## In Simple Terms

"Consumer demand" is the running total of items a subscriber has requested but not
yet received — essentially, the consumer's own stated capacity to keep up. A
well-behaved reactive consumer only requests as much as it can genuinely process in a
reasonable time, replenishing demand as it finishes work.

## Simple Example

```java
Flux.range(1, 100)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(1); // start conservatively
        }

        @Override
        protected void hookOnNext(Integer value) {
            slowlyProcess(value); // simulate a slow consumer
            request(1); // only ask for the next one once we're ready
        }
    });
```

This subscriber never has more than 1 outstanding item of demand at a time —
matching its own processing speed exactly.

## Why It Matters

Understanding consumer demand explains why some reactive pipelines "pull" data at a
controlled pace instead of being flooded — the subscriber is in the driver's seat,
declaring its own capacity, rather than the producer deciding unilaterally how fast
to push.
