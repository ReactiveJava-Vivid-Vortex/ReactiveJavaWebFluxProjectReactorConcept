# Demand Management

## In Simple Terms

**Demand** is simply how many items a subscriber has asked for that haven't
arrived yet. **Demand management** is just the publisher keeping track of that
number so it never sends more than it should.

Think of it as a running balance:

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
                request(5); // top up the demand every 5 items
            }
        }
    });
```

## Why It Matters

Keeping this balance right prevents two problems: a publisher racing way ahead
and piling up unsent data in memory, or a subscriber sitting idle because it
never asked for more. Operators like `.limitRate()` and `.prefetch()` in Project
Reactor are basically doing this bookkeeping for you, automatically.
