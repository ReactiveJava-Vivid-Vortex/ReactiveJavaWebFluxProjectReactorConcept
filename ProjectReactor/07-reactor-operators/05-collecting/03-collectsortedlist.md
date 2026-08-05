# collectSortedList()

## In Simple Terms

`.collectSortedList()` gathers all items from a `Flux` into a single `Mono<List<T>>`,
**sorted** either by their natural ordering (if `Comparable`) or by a custom
`Comparator` you provide. It's like `.collectList()` followed by `Collections.sort()`,
done in one step.

## Simple Example

```java
Flux.just(5, 3, 8, 1, 9)
    .collectSortedList()
    .subscribe(sorted -> System.out.println("Sorted: " + sorted));
```

Output:
```
Sorted: [1, 3, 5, 8, 9]
```

Using a custom comparator — sorting orders by total, descending:

```java
Flux.just(order1, order2, order3)
    .collectSortedList(Comparator.comparing(Order::getTotal).reversed())
    .subscribe(sortedOrders -> System.out.println(sortedOrders));
```

## Why It Matters

`.collectSortedList()` is convenient when you need a fully sorted result at the end
of a stream — e.g., generating a leaderboard, a top-N report, or any UI list that
must be presented in a specific order — without a separate manual sort step after
collecting.
