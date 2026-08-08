# takeWhile()

## In Simple Terms

`.takeWhile()` keeps letting items through as long as they pass a condition
— the moment one item fails, it stops for good, even if later items would
have passed. Think of it like walking down a line of people checking IDs:
as soon as you find someone underage, you stop checking the rest of the
line, even if some of them further back would have been fine.

## Simple Example

```java
Flux.just(1, 2, 3, 10, 4, 5)
    .takeWhile(n -> n < 5)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 1
Got: 2
Got: 3
```

The stream stops right at `10` (since `10 < 5` fails), and it never even
bothers looking at `4` or `5` — even though those would have passed the
check. Once `takeWhile` hits a failure, it's done.

## Why It Matters

`.takeWhile()` is perfect for streams that should stop the moment you hit a
"that's enough" signal — like reading lines from a file until you hit a
blank one, or going through a sorted list of transactions until you reach
one that's too old to care about.
