# doOnComplete()

## In Simple Terms

`.doOnComplete()` runs something only when the stream finishes the good
way — everything went fine, no errors. If the stream fails or gets
cancelled instead, this never fires. That makes it different from
`.doFinally()`, which fires no matter how things end.

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

Compare that with an error case, where `doOnComplete()` simply never runs:

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

`.doOnComplete()` is the right spot for logic that should only run on a
clean finish — marking a batch job "done successfully," sending a
completion notification, committing a transaction — as opposed to cleanup
work that has to happen regardless of outcome (that belongs in
`.doFinally()`).
