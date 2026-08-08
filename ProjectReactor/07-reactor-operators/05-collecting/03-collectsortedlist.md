# collectSortedList()

## In Simple Terms

`.collectSortedList()` does the same job as `.collectList()`, but it also
sorts everything before handing it back — either using the items' natural
order, or an ordering rule you give it. It's `.collectList()` plus sorting,
done as one step instead of two.

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

Using a custom sort order — orders by total, highest first:

```java
Flux.just(order1, order2, order3)
    .collectSortedList(Comparator.comparing(Order::getTotal).reversed())
    .subscribe(sortedOrders -> System.out.println(sortedOrders));
```

## Why It Matters

`.collectSortedList()` saves you a step whenever you need a fully sorted
result at the end of a stream — building a leaderboard, a top-N report, or
any list that has to appear in a particular order — instead of collecting
first and sorting separately afterward.
