# handle()

## In Simple Terms

`.handle()` is a Swiss-army-knife operator: for every item, you get to
decide — keep it as-is, change it into something else, throw it away
entirely, or raise an error — all in one spot. It's basically `.filter()`
and `.map()` merged into a single decision point.

## Simple Example

```java
Flux.just(1, 2, 3, 4, 5, 6)
    .handle((n, sink) -> {
        if (n % 2 == 0) {
            sink.next(n * 10); // keep it, transformed
        }
        // odd numbers: do nothing, so they just vanish
    })
    .subscribe(value -> System.out.println("Got: " + value));
```

Output:
```
Got: 20
Got: 40
Got: 60
```

This does the exact same job as `.filter(n -> n % 2 == 0).map(n -> n * 10)`,
just packed into one operator instead of two.

## Why It Matters

Reach for `.handle()` when deciding "should I keep this?" and "how should I
change it?" are really the same decision based on the same check — it saves
you from chaining separate `.filter()` and `.map()` calls. It's also handy
when you want to raise a custom error partway through, based on something
about a specific item.
