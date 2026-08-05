# firstWithValue()

## In Simple Terms

`Flux.firstWithValue(source1, source2, ...)` is similar to `firstWithSignal()`, but
it specifically waits for the first source to emit an **actual value** (not just any
signal). If a source errors or completes empty first, it's ignored in favor of
whichever source produces a real value first (as long as at least one eventually
does).

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

Even though `cache` "finished" first (with an empty result), `firstWithValue()`
correctly waits for `database` since it's the one that actually produced a value.

## firstWithSignal() vs firstWithValue()

| Scenario                                | firstWithSignal()             | firstWithValue()                |
|-------------------------------------------|----------------------------------|------------------------------------|
| One source completes empty quickly        | Wins immediately (any signal counts) | Ignored, waits for a real value  |
| One source errors quickly                 | Wins immediately (propagates error)  | Ignored (unless ALL sources fail)|

## Why It Matters

`.firstWithValue()` is the safer choice when racing sources that might legitimately
return "nothing" (like a cache miss) — you don't want an empty cache result to "win"
the race and short-circuit a database call that would have returned real data.
