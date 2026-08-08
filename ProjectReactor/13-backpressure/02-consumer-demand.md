# Consumer Demand

## In Simple Terms

"Consumer demand" is just the running count of items a subscriber has asked
for but not yet gotten — basically, how much work the consumer says it can
currently handle. A well-behaved consumer only asks for as much as it can
genuinely keep up with, topping up its request as it finishes each batch.

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

This subscriber never has more than 1 item outstanding at a time — always
asking for just as much as it can actually chew.

## Why It Matters

Understanding consumer demand explains why some pipelines pull data at a
steady, controlled pace instead of getting flooded — the consumer is in the
driver's seat here, declaring its own capacity, rather than the producer
deciding unilaterally how fast to push things at it.
