# skip()

## In Simple Terms

`.skip(n)` throws away the first `n` items and lets everything after that
through — like fast-forwarding past the opening credits of a movie. There's
also a time-based version, `.skip(Duration)`, that throws away whatever
shows up during an initial time window.

## Simple Example

```java
Flux.range(1, 10)
    .skip(3)
    .subscribe(n -> System.out.println("Got: " + n));
```

Output:
```
Got: 4
Got: 5
Got: 6
Got: 7
Got: 8
Got: 9
Got: 10
```

Often paired with `.take()` to grab a "page" out of the middle of a list:

```java
// "page 2" of size 5: skip the first 5, take the next 5
Flux.range(1, 20)
    .skip(5)
    .take(5)
    .subscribe(n -> System.out.println("Page item: " + n));
```

## Why It Matters

`.skip()` is a simple way to ignore leading items you don't care about —
skipping a header row in a data feed, or building basic pagination when you
combine it with `.take()`.
