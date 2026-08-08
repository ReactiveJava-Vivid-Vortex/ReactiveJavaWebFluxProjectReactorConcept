# mergeSequential()

## In Simple Terms

`Flux.mergeSequential()` gets you the best of both `merge()` and
`concat()`: it kicks off all sources at the same time (so they all start
working right away, just like `merge()`), but it still hands you the
results back in the original source order (like `concat()`) — quietly
holding onto faster results until it's their proper turn.

## Simple Example

```java
Flux<String> fast = Flux.just("A1", "A2").delayElements(Duration.ofMillis(50));
Flux<String> slow = Flux.just("B1", "B2").delayElements(Duration.ofMillis(200));

Flux.mergeSequential(fast, slow)
    .subscribe(item -> System.out.println("Got: " + item));
```

Even though `fast` finishes well before `slow`, the output order is
guaranteed:
```
Got: A1
Got: A2
Got: B1
Got: B2
```

Both sources started working concurrently the moment `mergeSequential`
subscribed — `fast`'s results are just held back internally until `slow`
has had its turn.

## Why It Matters

`.mergeSequential()` is the answer when you need predictable, ordered
output but still want all the underlying work to start at once for speed —
like calling several APIs in parallel but needing to show or process their
results in a fixed, known order.
