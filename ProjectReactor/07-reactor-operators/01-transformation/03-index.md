# index()

## In Simple Terms

`.index()` sticks a position number on every item as it goes by — like a
deli-counter ticket machine, except instead of "take a number," it's "here's
your number, now go." The first item gets 0, the second gets 1, and so on.
Each item comes out paired as (number, item).

## Simple Example

```java
Flux.just("Apple", "Banana", "Cherry")
    .index()
    .subscribe(tuple -> System.out.println(tuple.getT1() + ": " + tuple.getT2()));
```

Output:
```
0: Apple
1: Banana
2: Cherry
```

You can also format it however you like instead of getting a raw pair back:

```java
Flux.just("A", "B", "C")
    .index((idx, value) -> "Item #" + idx + " = " + value)
    .subscribe(System.out::println);
```

Output:
```
Item #0 = A
Item #1 = B
Item #2 = C
```

## Why It Matters

Whenever you need to know "which one is this" — for logging like "processing
item 5 of N," numbering rows in a report, or anything positional — `.index()`
gives you that number for free, instead of you having to keep a counter
variable and manually increment it yourself.
