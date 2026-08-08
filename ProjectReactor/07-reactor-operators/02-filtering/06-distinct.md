# distinct()

## In Simple Terms

`.distinct()` weeds out repeats — the first time it sees a value, it lets it
through; if that same value shows up again later, it gets quietly dropped.
It works the same way as removing duplicates from a list, just applied to a
stream as it flows.

## Simple Example

```java
Flux.just(1, 2, 2, 3, 1, 4, 3)
    .distinct()
    .subscribe(n -> System.out.println("Unique: " + n));
```

Output:
```
Unique: 1
Unique: 2
Unique: 3
Unique: 4
```

You can also tell it what counts as "the same" — for example, deduplicating
orders by customer instead of comparing whole objects:

```java
Flux.just(order1, order2, order3)
    .distinct(Order::getCustomerId) // dedupe based on this key
    .subscribe(order -> System.out.println("First order per customer: " + order));
```

**Good to know:** `.distinct()` has to remember every unique value it has
seen so far, so it can spot repeats later. That means it's not a great fit
for a stream that runs forever with lots of different values — its memory
usage just keeps growing.

## Why It Matters

`.distinct()` is a quick, clean way to remove duplicates — handy when
combining data from more than one source, like a cache and a database, where
the same record might show up twice.
