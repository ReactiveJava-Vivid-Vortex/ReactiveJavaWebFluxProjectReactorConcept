# groupBy()

## In Simple Terms

`.groupBy()` sorts a stream into separate sub-streams based on a key you
give it — much like SQL's `GROUP BY`, or sorting mail into different
mailboxes by recipient. Each sub-stream (a `GroupedFlux`) knows its own key
and only contains the items that belong to that group.

## Simple Example

```java
Flux.just("apple", "banana", "avocado", "blueberry", "cherry")
    .groupBy(word -> word.charAt(0)) // group by first letter
    .flatMap(groupedFlux ->
        groupedFlux.collectList()
            .map(list -> groupedFlux.key() + ": " + list)
    )
    .subscribe(System.out::println);
```

Output (order between groups may vary since groups are processed concurrently):
```
a: [apple, avocado]
b: [banana, blueberry]
c: [cherry]
```

A practical example — grouping orders by customer:

```java
orderFlux
    .groupBy(Order::getCustomerId)
    .flatMap(groupedFlux ->
        groupedFlux.count().map(count -> groupedFlux.key() + " placed " + count + " orders")
    )
    .subscribe(System.out::println);
```

**Watch out for this:** every `GroupedFlux` needs to actually be subscribed
to (usually through `.flatMap()`) — if you ignore a group without draining
it, its items just pile up in memory, since nothing is asking for them.

## Why It Matters

`.groupBy()` is essential for splitting up a stream by category — grouping
Kafka messages by customer for per-customer handling, or splitting a log
stream by severity level for different processing — all done reactively,
without you having to hand-manage a `Map<Key, List<T>>` yourself.
