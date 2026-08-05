# takeWhile()

## In Simple Terms

`.takeWhile(predicate)` lets items through **only as long as the predicate is
true**, and stops (completes) the moment the predicate first returns `false` — even
if later items would have satisfied it again. It checks the condition on the item
itself before deciding to pass it through.

## Simple Example

```java
Flux.just(1, 2, 3, 10, 4, 5)
    .takeWhile(n -> n < 5)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 1
Got: 2
Got: 3
```

Notice the stream stops at `10` (since `10 < 5` is false) and **never even looks at**
`4` or `5`, even though they would have passed the condition — `takeWhile` stops
permanently at the first failure.

## Why It Matters

`.takeWhile()` is perfect for streams that should stop as soon as a "sentinel"
condition is hit — e.g., reading lines from a stream until an empty line marks the
end, or processing a sorted list of transactions until you hit one older than a cutoff
date.
