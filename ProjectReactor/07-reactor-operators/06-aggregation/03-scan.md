# scan()

## In Simple Terms

`.scan()` is `.reduce()`'s more talkative cousin: instead of only giving you
the final snowball at the end, it hands you the snowball after *every single
roll* — a running total that updates with each item, streamed out as it
grows.

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

Notice that unlike `.reduce()` (which would only print `15`), `.scan()`
shows you every step on the way there.

With a starting value:

```java
Flux.just(1, 2, 3)
    .scan(100, (runningTotal, next) -> runningTotal + next)
    .subscribe(System.out::println);
// 100, 101, 103, 106
```

## Why It Matters

`.scan()` is perfect for anything that updates live — a running balance
shown on a dashboard as transactions come in, or a leaderboard score that
ticks up point by point — anywhere you want to watch the total change in
real time, not just see the final number at the end.
