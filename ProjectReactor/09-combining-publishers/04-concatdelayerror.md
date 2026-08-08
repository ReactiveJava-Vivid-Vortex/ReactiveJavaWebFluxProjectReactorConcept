# concatDelayError()

## In Simple Terms

`Flux.concatDelayError()` behaves like `.concat()`, except if one source
fails, it doesn't stop the whole show right away — it keeps going through
the remaining sources first, and only reports the failure at the very end,
after everything that *could* succeed has had its turn.

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

Compare with plain `.concat()`, where `first`'s error would immediately end
everything — `second`'s items (3, 4) would never show up at all.

## Why It Matters

`.concatDelayError()` is useful for batch jobs where you want to try
*everything* (like sending out 10 emails) even if some fail along the way,
and only find out about the failures at the end — rather than giving up
early and leaving the rest of the work undone.
