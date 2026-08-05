# Demand Management

## In Simple Terms

**Demand** is the number of items a subscriber has asked for (via `request(n)`) that
haven't been delivered yet. **Demand management** is the bookkeeping the publisher
does to make sure it never sends more items than the outstanding demand allows.

Think of demand as a running balance:

```
Subscriber calls request(5)   -> demand = 5
Publisher sends onNext() x3   -> demand = 2 (3 used up)
Subscriber calls request(2)   -> demand = 4
Publisher sends onNext() x4   -> demand = 0 (must stop until asked again)
```

## Simple Example

```java
Flux.range(1, 100)
    .subscribe(new BaseSubscriber<Integer>() {
        int received = 0;

        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(5); // initial demand: 5
        }

        @Override
        protected void hookOnNext(Integer value) {
            received++;
            System.out.println("Got: " + value);
            if (received % 5 == 0) {
                request(5); // replenish demand every 5 items
            }
        }
    });
```

## Why It Matters

Good demand management is what prevents both **overproduction** (publisher racing
ahead and building up an unbounded buffer) and **underutilization** (subscriber
starving because it never asks for more). Operators like `.limitRate()` and
`.prefetch()` in Project Reactor are essentially fine-tuning demand management for
you automatically.
