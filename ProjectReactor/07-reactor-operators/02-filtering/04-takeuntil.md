# takeUntil()

## In Simple Terms

`.takeUntil(predicate)` is similar to `.takeWhile()`, but with a key difference:
it includes the item that **triggers** the stop condition, then completes.
`takeWhile` **excludes** the failing item; `takeUntil` **includes** the matching
item.

## Simple Example

```java
Flux.just(1, 2, 3, 10, 4, 5)
    .takeUntil(n -> n > 5)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 1
Got: 2
Got: 3
Got: 10
```

Notice `10` **is included** (it's the item that satisfies `n > 5`, so `takeUntil`
stops right after emitting it) — and `4`, `5` are never processed.

## Comparison with takeWhile()

```
takeWhile(n -> n < 5)   on [1,2,3,10,4,5]  ->  1, 2, 3         (stops BEFORE the failing item)
takeUntil(n -> n > 5)   on [1,2,3,10,4,5]  ->  1, 2, 3, 10      (stops AFTER the matching item)
```

## Why It Matters

`.takeUntil()` is useful when the triggering item itself is meaningful and should be
included in the result — e.g., reading events "until and including" a shutdown
signal, or processing a batch "up to and including" an error marker.
