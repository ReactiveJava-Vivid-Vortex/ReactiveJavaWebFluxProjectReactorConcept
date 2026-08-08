# Cold Publishers

## In Simple Terms

A **cold publisher** starts its work **fresh, from zero, for every single
subscriber.** Think Netflix — every viewer who presses play starts the movie from
the very beginning, no matter when they joined.

Most of the things you create in Reactor — `Flux.just()`, `Flux.fromIterable()`,
a database query, an HTTP call — are cold by default.

## Simple Example

```java
Flux<Long> coldFlux = Flux.just(System.currentTimeMillis());

coldFlux.subscribe(time -> System.out.println("Subscriber 1: " + time));

try { Thread.sleep(2000); } catch (InterruptedException e) {}

coldFlux.subscribe(time -> System.out.println("Subscriber 2: " + time));
```

Output (different timestamps — each subscriber triggered its own fresh run):
```
Subscriber 1: 1732000000000
Subscriber 2: 1732000002000
```

It's the exact same `coldFlux` object both times, but subscribing twice runs
`System.currentTimeMillis()` separately, once per subscriber.

## Why It Matters

Cold is the default, and it's usually what you want — a database query should
fetch fresh data for each subscriber, not reuse one cached run. When you *do*
want subscribers to share the exact same live run (like tuning into a broadcast
instead of each getting their own replay), you turn a cold publisher into a
**hot** one using `.share()` or `.publish()` — covered in the Hot & Cold
Publishers topic.
