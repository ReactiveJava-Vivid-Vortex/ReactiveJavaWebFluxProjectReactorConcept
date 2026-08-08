# onErrorComplete()

## In Simple Terms

`.onErrorComplete()` catches an error and just quietly wraps up the stream
— as if it finished normally with nothing more to say — instead of passing
the error along or offering a fallback value. It basically says "if this
fails, just stop, no drama."

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

You can also limit it to specific exception types:

```java
someFlux.onErrorComplete(SpecificException.class);
```

## Why It Matters

`.onErrorComplete()` fits when an error really just means "nothing else to
report" rather than "something's genuinely wrong here" — like an optional
enrichment step where failing just means skipping some extra data, not
failing the whole operation.
