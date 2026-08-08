# concatWith()

## In Simple Terms

`.concatWith()` does the exact same thing as `Flux.concat()` — play one
stream fully, then the next, in strict order — just written as a fluent
chain (`a.concatWith(b)`) instead of a static call (`Flux.concat(a, b)`).

## Simple Example

```java
Flux<String> morning = Flux.just("Breakfast", "Coffee");
Flux<String> afternoon = Flux.just("Lunch", "Meeting");

morning.concatWith(afternoon)
    .subscribe(System.out::println);
```

Output:
```
Breakfast
Coffee
Lunch
Meeting
```

You can chain several of these together:

```java
Flux.just("Step 1")
    .concatWith(Flux.just("Step 2"))
    .concatWith(Flux.just("Step 3"))
    .subscribe(System.out::println);
// Step 1, Step 2, Step 3 - always in this order
```

## Why It Matters

`.concatWith()` just reads more naturally in a fluent chain than
`Flux.concat(a, b)` does. It's commonly used to tack on a "final summary"
item, or to chain together the sequential steps of a multi-part operation
where the order absolutely has to stay fixed.
