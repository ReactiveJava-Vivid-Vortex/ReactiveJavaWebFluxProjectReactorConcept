# takeUntil()

## In Simple Terms

`.takeUntil()` is almost the same idea as `.takeWhile()`, but with one
difference that trips people up: it **includes** the item that triggers the
stop, instead of leaving it out. `takeWhile` stops right *before* the bad
item; `takeUntil` stops right *after* it.

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

`10` **is included** here — it's the item that made the condition true, so
`takeUntil` lets it through and then stops. `4` and `5` never get a look-in.

## Comparison with takeWhile()

```
takeWhile(n -> n < 5)   on [1,2,3,10,4,5]  ->  1, 2, 3         (stops BEFORE the failing item)
takeUntil(n -> n > 5)   on [1,2,3,10,4,5]  ->  1, 2, 3, 10      (stops AFTER the matching item)
```

## Why It Matters

Use `.takeUntil()` when the item that triggers the stop is itself important
and you want to keep it — like reading events "up to and including" a
shutdown signal, or processing a batch "through" the item that marks an
error.
