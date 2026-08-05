# Overflow Handling

## In Simple Terms

"Overflow" happens when a producer emits items faster than the consumer has
requested, and there's nowhere safe to put the excess. Reactive Streams sources
(especially manually created ones like `Flux.create()`) need an explicit
**overflow strategy** to decide what to do in that situation.

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

Choosing the right overflow strategy is a real design decision: `BUFFER` is safest
for correctness but risks memory issues if truly unbounded; `LATEST`/`DROP` trade
completeness for bounded memory (ideal for live data like sensor readings where only
the newest value matters); `ERROR` surfaces the mismatch loudly instead of silently
dropping data.
