# Producer Speed

## In Simple Terms

"Producer speed" just means how fast a source can crank out new items. When
a producer is way faster than the consumer, backpressure exists to stop
that speed mismatch from piling up unbounded — the producer has to be told,
and has to respect, how much the consumer can actually handle right now.

## Simple Example

```java
Flux<Integer> fastProducer = Flux.range(1, 1_000_000); // can produce instantly

fastProducer
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(5); // artificially slow ourselves down as the consumer
        }

        @Override
        protected void hookOnNext(Integer value) {
            try { Thread.sleep(100); } catch (InterruptedException ignored) {} // slow processing
            System.out.println("Processed: " + value);
            request(1);
        }
    });
```

Even though `Flux.range()` could pump out a million items almost instantly,
the subscriber's careful `request()` calls keep it in check — the source
only produces as much as has actually been asked for.

## Why It Matters

Pair a fast producer with a slow consumer and skip backpressure entirely,
and you get unbounded buffering — eventually crashing with an
`OutOfMemoryError`. The demand model in Reactive Streams makes sure
producer speed always stays tied to what the consumer can genuinely handle
— this is the whole problem backpressure exists to solve.
