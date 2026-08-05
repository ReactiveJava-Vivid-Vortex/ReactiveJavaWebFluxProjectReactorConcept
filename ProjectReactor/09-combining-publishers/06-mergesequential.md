# mergeSequential()

## In Simple Terms

`Flux.mergeSequential(source1, source2, ...)` is a hybrid: it **subscribes to all
sources concurrently** (like `merge()`, so they all start working in parallel right
away), but it **emits their results in source order** (like `concat()`) — buffering
faster sources' output until it's their turn.

## Simple Example

```java
Flux<String> fast = Flux.just("A1", "A2").delayElements(Duration.ofMillis(50));
Flux<String> slow = Flux.just("B1", "B2").delayElements(Duration.ofMillis(200));

Flux.mergeSequential(fast, slow)
    .subscribe(item -> System.out.println("Got: " + item));
```

Even though `fast` finishes well before `slow`, the output order is guaranteed:
```
Got: A1
Got: A2
Got: B1
Got: B2
```

Both sources started working (concurrently) as soon as `mergeSequential` subscribed
— `fast`'s results are just held back internally until `slow`'s turn is over.

## Why It Matters

`.mergeSequential()` gives you the best of both worlds when you need **deterministic
output ordering** but still want to kick off all the underlying work concurrently for
speed — e.g., calling multiple APIs in parallel but needing to display/process their
results in a fixed, predictable order.
