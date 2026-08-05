# startWith()

## In Simple Terms

`.startWith(items...)` prepends one or more values (or another `Publisher`) **before**
the original sequence starts — the extra items are emitted first, then the original
`Flux`'s own items follow.

## Simple Example

```java
Flux.just("Banana", "Cherry")
    .startWith("Apple")
    .subscribe(System.out::println);
```

Output:
```
Apple
Banana
Cherry
```

You can also prepend an entire other `Publisher`:

```java
Flux<String> header = Flux.just("--- Report Start ---");
Flux<String> data = Flux.just("Row 1", "Row 2");

data.startWith(header)
    .subscribe(System.out::println);
```

Output:
```
--- Report Start ---
Row 1
Row 2
```

## Why It Matters

`.startWith()` is a simple, readable way to inject a header, a default/sentinel
value, or a cached "last known value" before a live stream begins — a common need
when building UIs that should show something immediately, followed by live updates.
