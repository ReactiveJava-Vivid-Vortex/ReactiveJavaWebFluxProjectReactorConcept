# reduce()

## In Simple Terms

`.reduce()` boils down every item in a `Flux` into one single final answer.
It works like a snowball rolling downhill: it keeps a "running total," and
for each new item, it combines the running total with that item to get a
new running total. When the stream ends, you only get the very last
snowball — the final result — as a `Mono`.

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

You can also give it a starting point instead of using the first item:

```java
Flux.just(1, 2, 3, 4, 5)
    .reduce(100, (sum, next) -> sum + next) // start from 100 instead of the first item
    .subscribe(total -> System.out.println("Sum: " + total));
// Sum: 115
```

A practical example — adding up order totals:

```java
orderFlux
    .map(Order::getTotal)
    .reduce(0.0, Double::sum)
    .subscribe(grandTotal -> System.out.println("Grand total: " + grandTotal));
```

## Why It Matters

`.reduce()` is your general-purpose tool for turning "lots of values" into
"one summary value" — totals, products, joined strings, a manually-tracked
max or min, or any custom running calculation — whenever all you care about
is the final number, not every step along the way.
