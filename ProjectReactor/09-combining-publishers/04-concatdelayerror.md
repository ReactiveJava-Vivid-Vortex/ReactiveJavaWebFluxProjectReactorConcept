# concatDelayError()

## In Simple Terms

`Flux.concatDelayError(source1, source2, ...)` is like `.concat()`, but if one of the
sources errors, it **doesn't stop immediately** — it continues processing the
remaining sources first, and only surfaces the error at the very end, after
everything that could succeed has been emitted.

## Simple Example

```java
Flux<Integer> first = Flux.just(1, 2).concatWith(Flux.error(new RuntimeException("First failed")));
Flux<Integer> second = Flux.just(3, 4);

Flux.concatDelayError(first, second)
    .subscribe(
        n -> System.out.println("Got: " + n),
        error -> System.out.println("Error (at the end): " + error.getMessage())
    );
```

Output:
```
Got: 1
Got: 2
Got: 3
Got: 4
Error (at the end): First failed
```

Compare with regular `.concat()`, where the error from `first` would immediately
terminate the whole sequence — `second`'s items (3, 4) would **never** be emitted.

## Why It Matters

`.concatDelayError()` is useful in batch-processing scenarios where you want to
attempt **all** operations (e.g., sending 10 emails) even if some fail partway
through, and only report the failure(s) after everything that could succeed has been
given a chance to run — rather than aborting early and leaving later, unrelated work
undone.
