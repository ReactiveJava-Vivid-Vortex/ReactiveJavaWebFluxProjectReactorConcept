# scan()

## In Simple Terms

`.scan(accumulator)` is like `.reduce()`, but instead of emitting only the final
result, it emits **every intermediate running total** as it goes — so you get a
`Flux` of "running sum so far," one value per input item, rather than just one final
`Mono`.

## Simple Example

```java
Flux.just(1, 2, 3, 4, 5)
    .scan((runningTotal, next) -> runningTotal + next)
    .subscribe(total -> System.out.println("Running total: " + total));
```

Output:
```
Running total: 1
Running total: 3
Running total: 6
Running total: 10
Running total: 15
```

Notice unlike `.reduce()` (which only prints `15`), `.scan()` shows every step along
the way.

With an initial seed value:

```java
Flux.just(1, 2, 3)
    .scan(100, (runningTotal, next) -> runningTotal + next)
    .subscribe(System.out::println);
// 100, 101, 103, 106
```

## Why It Matters

`.scan()` is perfect for live, incrementally-updating displays — e.g., a running
balance shown on a dashboard as transactions stream in, or a live leaderboard score
that updates with each new point scored — where you want to see every intermediate
state, not just the final one.
