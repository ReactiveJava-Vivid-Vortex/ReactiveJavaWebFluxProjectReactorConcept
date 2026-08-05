# concatWith()

## In Simple Terms

`.concatWith(other)` is the instance-method version of `Flux.concat()` — it appends
another publisher's items **after** the current one fully completes, preserving
strict ordering, just written as a fluent chain instead of a static method call.

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

Chaining multiple `.concatWith()` calls:

```java
Flux.just("Step 1")
    .concatWith(Flux.just("Step 2"))
    .concatWith(Flux.just("Step 3"))
    .subscribe(System.out::println);
// Step 1, Step 2, Step 3 - always in this order
```

## Why It Matters

`.concatWith()` reads more naturally in a fluent chain than `Flux.concat(a, b)`, and
is commonly used to append a "final summary" item, or chain together sequential
stages of a multi-part reactive operation where order strictly matters.
