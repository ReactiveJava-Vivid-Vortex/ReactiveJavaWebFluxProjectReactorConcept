# startWith()

## In Simple Terms

`.startWith()` sticks one or more extra items at the very front of a
stream, before anything else — like adding an opening line before the rest
of a story starts. The extra stuff comes out first, then the original
items follow in their normal order.

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

You can also prepend a whole other stream instead of just one value:

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

`.startWith()` is a clean way to add a header, a default value, or a
cached "last known value" before the live data shows up — useful for a UI
that should display *something* right away, followed by real updates as
they arrive.
