# Overflow Handling

## In Simple Terms

"Overflow" happens when a producer sends out more than the consumer asked
for, and there's nowhere safe to put the excess — like a mailbox getting
more letters than it can hold. Sources you build by hand (especially
`Flux.create()`) need an explicit overflow strategy to decide what happens
when that occurs.

## Simple Example

```java
Flux.create((FluxSink<Integer> sink) -> {
    for (int i = 0; i < 1000; i++) {
        sink.next(i); // emits regardless of downstream demand
    }
    sink.complete();
}, FluxSink.OverflowStrategy.BUFFER) // choose how to handle overflow
.subscribe(new BaseSubscriber<Integer>() {
    @Override
    protected void hookOnSubscribe(Subscription subscription) {
        request(1); // deliberately slow consumer
    }
    @Override
    protected void hookOnNext(Integer value) {
        System.out.println("Got: " + value);
        request(1);
    }
});
```

Available `OverflowStrategy` options:

| Strategy  | Behavior when demand is exceeded                          |
|-----------|--------------------------------------------------------------|
| `BUFFER`  | Queue excess items in memory (risk: unbounded growth)         |
| `DROP`    | Discard new items that exceed demand                          |
| `LATEST`  | Keep only the most recent item, discarding older excess ones   |
| `ERROR`   | Terminate the stream with an error if demand is exceeded       |
| `IGNORE`  | Do nothing special — may violate the spec if misused           |

## Why It Matters

Picking the right overflow strategy is a genuine design decision:
`BUFFER` is the safest choice for correctness, but risks memory trouble if
truly unbounded; `LATEST`/`DROP` trade completeness for keeping memory
bounded (great for live data like sensor readings where only the newest
value actually matters); `ERROR` makes the mismatch loud and obvious
instead of quietly dropping data.
