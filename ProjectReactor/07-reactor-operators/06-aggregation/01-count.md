# count()

## In Simple Terms

`.count()` waits for a `Flux` to complete, then emits a single `Mono<Long>` with the
**total number of items** it saw. It's the reactive equivalent of counting the size
of a list.

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

Combined with `.filter()` to count matching items:

```java
Flux.range(1, 100)
    .filter(n -> n % 7 == 0)
    .count()
    .subscribe(count -> System.out.println("Multiples of 7: " + count));
// Multiples of 7: 14
```

## Why It Matters

`.count()` is a simple, common building block for reporting/metrics — e.g., "how many
orders were processed today," or validating that an expected number of items came
through a pipeline before proceeding with the next step.
