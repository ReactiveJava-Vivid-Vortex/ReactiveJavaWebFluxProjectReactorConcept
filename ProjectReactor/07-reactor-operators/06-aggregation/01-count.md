# count()

## In Simple Terms

`.count()` waits for a `Flux` to finish and then tells you exactly how many
items went by — a single number, wrapped in a `Mono`. It's the same idea as
checking `list.size()`, just for a stream instead of a list.

## Simple Example

```java
Flux.just("a", "b", "c", "d")
    .count()
    .subscribe(total -> System.out.println("Total items: " + total));
```

Output:
```
Total items: 4
```

Paired with `.filter()` to count only the items that match something:

```java
Flux.range(1, 100)
    .filter(n -> n % 7 == 0)
    .count()
    .subscribe(count -> System.out.println("Multiples of 7: " + count));
// Multiples of 7: 14
```

## Why It Matters

`.count()` is a simple, everyday tool for reporting and metrics — things
like "how many orders came in today," or double-checking that the expected
number of items actually made it through a pipeline before moving on to the
next step.
