# Requesting Elements

## In Simple Terms

"Requesting elements" refers to the subscriber-side action of calling
`subscription.request(n)` to say how many items it's ready to receive next. This is
the core mechanic subscribers use to pace their own consumption, and it can happen
multiple times throughout the life of a subscription (not just once at the start).

## Simple Example

```java
Flux.range(1, 10)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            System.out.println("Requesting first batch of 3");
            request(3);
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println("Processing: " + value);
            if (value % 3 == 0) {
                System.out.println("Requesting next batch of 3");
                request(3);
            }
        }
    });
```

Output shows the subscriber pulling data in controlled batches of 3, rather than all
10 items being pushed at once:

```
Requesting first batch of 3
Processing: 1
Processing: 2
Processing: 3
Requesting next batch of 3
Processing: 4
Processing: 5
Processing: 6
Requesting next batch of 3
...
```

## Why It Matters

Requesting elements in controlled batches (instead of `Long.MAX_VALUE` all at once)
is how you build **flow-controlled** consumers — useful when downstream processing
(e.g., writing to a database, or calling a rate-limited API) is slower than the
upstream can produce data.
