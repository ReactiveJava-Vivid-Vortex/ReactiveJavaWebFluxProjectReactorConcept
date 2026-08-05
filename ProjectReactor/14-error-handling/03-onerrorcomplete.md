# onErrorComplete()

## In Simple Terms

`.onErrorComplete()` catches an error and simply **completes the stream silently**
(as if it had ended successfully with no more items), instead of propagating the
error or providing a fallback value. It effectively says "if this fails, just stop
quietly."

## Simple Example

```java
Flux.just(1, 2, 3)
    .map(n -> {
        if (n == 3) throw new RuntimeException("Simulated failure");
        return n;
    })
    .onErrorComplete()
    .subscribe(
        n -> System.out.println("Item: " + n),
        error -> System.out.println("This will NOT print"),
        () -> System.out.println("Completed (silently, due to error)")
    );
```

Output:
```
Item: 1
Item: 2
Completed (silently, due to error)
```

You can also restrict it to specific exception types:

```java
someFlux.onErrorComplete(SpecificException.class);
```

## Why It Matters

`.onErrorComplete()` is useful when an error genuinely means "there's nothing more to
report" rather than "something went wrong that the caller needs to know about" — for
example, an optional enrichment step where failure should just mean "skip this extra
data," not fail the whole operation.
