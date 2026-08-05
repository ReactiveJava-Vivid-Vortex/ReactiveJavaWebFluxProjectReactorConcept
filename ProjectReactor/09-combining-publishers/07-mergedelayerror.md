# mergeDelayError()

## In Simple Terms

`Flux.mergeDelayError(concurrency, source1, source2, ...)` is like `merge()`, but if
any source errors, the error is **delayed** until all other sources have completed
(successfully or not) — so a failure in one source doesn't cut off the results still
arriving from the others.

## Simple Example

```java
Flux<String> ok = Flux.just("A", "B", "C").delayElements(Duration.ofMillis(50));
Flux<String> failing = Flux.<String>error(new RuntimeException("Service down"))
    .delaySubscription(Duration.ofMillis(75));

Flux.mergeDelayError(2, ok, failing)
    .subscribe(
        item -> System.out.println("Got: " + item),
        error -> System.out.println("Error at the end: " + error.getMessage())
    );
```

Output:
```
Got: A
Got: B
Got: C
Error at the end: Service down
```

With plain `.merge()`, the error from `failing` would immediately terminate the
whole stream, likely **before** `ok` had a chance to emit all its items.

## Why It Matters

`.mergeDelayError()` is important when combining results from multiple independent,
unreliable sources (e.g., aggregating data from several microservices) where one
service failing shouldn't prevent you from still getting the successful results from
the others — you find out about the failure only after everything else has had its
chance to complete.
