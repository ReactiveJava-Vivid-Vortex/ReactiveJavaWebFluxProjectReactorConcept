# reduce()

## In Simple Terms

`.reduce(accumulator)` combines all items in a `Flux` into a **single final value**,
by repeatedly applying a function that takes the "running total so far" and the
"next item," producing a new running total. Only the final result is emitted (as a
`Mono`), once the stream completes.

## Simple Example

```java
Flux.just(1, 2, 3, 4, 5)
    .reduce((sum, next) -> sum + next)
    .subscribe(total -> System.out.println("Sum: " + total));
```

Output:
```
Sum: 15
```

With an explicit initial/seed value (using the overload that takes a starting
value):

```java
Flux.just(1, 2, 3, 4, 5)
    .reduce(100, (sum, next) -> sum + next) // start from 100 instead of the first item
    .subscribe(total -> System.out.println("Sum: " + total));
// Sum: 115
```

A practical example — computing a running total of order values:

```java
orderFlux
    .map(Order::getTotal)
    .reduce(0.0, Double::sum)
    .subscribe(grandTotal -> System.out.println("Grand total: " + grandTotal));
```

## Why It Matters

`.reduce()` is the general-purpose tool for turning "many values" into "one summary
value" — sums, products, string concatenation, finding a max/min manually, or
building up any custom aggregate — whenever you need just the final result, not each
intermediate step.
