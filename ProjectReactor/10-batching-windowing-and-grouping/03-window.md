# window()

## In Simple Terms

`.window(n)` is similar to `.buffer(n)`, but instead of collecting items into a
`List`, it groups them into **separate `Flux` sub-streams** ("windows") of size `n`.
Each window is itself a `Flux<T>` that you can process reactively (e.g., with further
operators), rather than getting one big collected `List` all at once.

## Simple Example

```java
Flux.range(1, 9)
    .window(3)
    .flatMap(windowFlux -> windowFlux.collectList())
    .subscribe(batch -> System.out.println("Window: " + batch));
```

Output:
```
Window: [1, 2, 3]
Window: [4, 5, 6]
Window: [7, 8, 9]
```

The key difference from `.buffer()` shows up when you process each window reactively,
without collecting it into a list first:

```java
Flux.range(1, 9)
    .window(3)
    .flatMap(windowFlux -> windowFlux.reduce(0, Integer::sum)) // sum each window
    .subscribe(sum -> System.out.println("Window sum: " + sum));
// Window sum: 6  (1+2+3)
// Window sum: 15 (4+5+6)
// Window sum: 24 (7+8+9)
```

## Why It Matters

`.window()` gives you more flexibility than `.buffer()` because each window remains a
**stream**, not a fully materialized `List` — this matters when windows might be very
large (avoiding loading a huge list into memory at once) or when you want to apply
reactive operators (like `.reduce()`, `.filter()`) to each window individually.
