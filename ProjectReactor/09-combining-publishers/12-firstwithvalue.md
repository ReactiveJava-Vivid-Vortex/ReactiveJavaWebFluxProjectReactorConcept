# firstWithValue()

## In Simple Terms

`Flux.firstWithValue()` is like `firstWithSignal()`, but pickier — it
specifically waits for the first source to hand back an actual value, not
just any response. If a source errors out or finishes with nothing, it's
skipped in favor of whichever source actually produces real data first (as
long as one eventually does).

## Simple Example

```java
Mono<String> cache = Mono.<String>empty(); // cache miss, completes empty
Mono<String> database = Mono.just("Data from DB").delayElement(Duration.ofMillis(50));

Mono.firstWithValue(cache, database)
    .subscribe(result -> System.out.println("Got: " + result));
```

Output:
```
Got: Data from DB
```

Even though `cache` technically "finished" first (with nothing to show),
`firstWithValue()` correctly waits for `database` since that's the one that
actually gave a real answer.

## firstWithSignal() vs firstWithValue()

| Scenario                                | firstWithSignal()             | firstWithValue()                |
|-------------------------------------------|----------------------------------|------------------------------------|
| One source completes empty quickly        | Wins immediately (any signal counts) | Ignored, waits for a real value  |
| One source errors quickly                 | Wins immediately (propagates error)  | Ignored (unless ALL sources fail)|

## Why It Matters

`.firstWithValue()` is the safer pick when racing sources that might
legitimately come back with "nothing" — like a cache miss. You don't want an
empty cache result to "win" the race and short-circuit a database call that
would have actually returned real data.
