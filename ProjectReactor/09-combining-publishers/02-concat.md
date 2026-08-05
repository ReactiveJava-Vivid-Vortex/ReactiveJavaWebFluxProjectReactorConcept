# concat()

## In Simple Terms

`Flux.concat(source1, source2, ...)` combines multiple publishers **sequentially** —
it fully exhausts the first source (waits for its `onComplete()`) before starting to
subscribe to the next one. The order is strictly preserved, and no interleaving
happens between sources.

## Simple Example

```java
Flux<Integer> first = Flux.just(1, 2, 3);
Flux<Integer> second = Flux.just(4, 5, 6);

Flux.concat(first, second)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output (strictly in order — first's items, then second's items):
```
Got: 1
Got: 2
Got: 3
Got: 4
Got: 5
Got: 6
```

Even if `second` could produce values faster, `concat()` will not start it until
`first` fully completes.

## Why It Matters

`.concat()` (and its instance-method sibling `.concatWith()`) is the right tool
whenever **order matters** and sources must not interleave — e.g., processing a
"setup" stream completely before starting a "main" stream, or combining paginated
results from multiple sources in a specific sequence.
