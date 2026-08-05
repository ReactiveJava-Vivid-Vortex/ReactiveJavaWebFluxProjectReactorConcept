# doOnComplete()

## In Simple Terms

`.doOnComplete(runnable)` runs a side effect **only when the stream completes
successfully** (i.e., `onComplete()` fires) — it does NOT run if the stream errors
out or is cancelled. This makes it different from `.doFinally()`, which runs in all
three cases.

## Simple Example

```java
Flux.just(1, 2, 3)
    .doOnComplete(() -> System.out.println("All items processed successfully!"))
    .subscribe(n -> System.out.println("Item: " + n));
```

Output:
```
Item: 1
Item: 2
Item: 3
All items processed successfully!
```

Contrast with an error case — `doOnComplete()` never fires:

```java
Flux.just(1, 2, 0)
    .map(n -> 10 / n)
    .doOnComplete(() -> System.out.println("This will NOT print"))
    .subscribe(
        n -> System.out.println("Item: " + n),
        error -> System.out.println("Error: " + error.getMessage())
    );
```

## Why It Matters

`.doOnComplete()` is the right hook for logic that should run **only on success** —
e.g., marking a batch job as "finished successfully," sending a completion
notification, or committing a transaction — distinct from cleanup logic that should
run regardless of outcome (which belongs in `.doFinally()`).
