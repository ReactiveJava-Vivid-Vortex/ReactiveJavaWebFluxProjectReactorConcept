# concat()

## In Simple Terms

`Flux.concat()` runs multiple streams one after another, like a playlist —
it plays the first one all the way through before starting the second one.
Order is always preserved, and nothing ever plays at the same time as
anything else.

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

Even if `second` could go faster, `concat()` waits until `first` is
completely done before it even starts it.

## Why It Matters

`.concat()` (and its chainable sibling `.concatWith()`) is the right tool
whenever order matters and things shouldn't run at the same time — like
finishing a "setup" step completely before starting the "main" step, or
combining paginated results from several sources in a specific sequence.
