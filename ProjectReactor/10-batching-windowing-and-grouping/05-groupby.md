# groupBy()

## In Simple Terms

`.groupBy(keySelector)` splits a `Flux` into multiple sub-streams (`GroupedFlux`),
one per distinct key — similar to SQL's `GROUP BY`. Each `GroupedFlux` carries its
key (accessible via `.key()`) and contains only the items belonging to that group.

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

**Important gotcha:** each `GroupedFlux` must be subscribed to (usually via
`.flatMap()`) — if you ignore a group without consuming it, its buffered items can
build up in memory, since nothing is requesting/draining them.

## Why It Matters

`.groupBy()` is essential for stream-partitioning use cases — e.g., grouping Kafka
messages by customer ID for per-customer processing, or splitting a log stream by
severity level for different downstream handling — all done reactively, without
manually maintaining a `Map<Key, List<T>>`.
