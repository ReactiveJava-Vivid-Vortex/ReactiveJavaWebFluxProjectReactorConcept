# filter()

## In Simple Terms

`.filter()` is a bouncer at the door: it checks each item against a rule you
give it, and only the ones that pass the check get let through. Anything
that fails is quietly turned away — no error, no fuss, it just doesn't make
it into the stream.

## Simple Example

```java
Flux.range(1, 10)
    .filter(n -> n % 2 == 0)
    .subscribe(even -> System.out.println("Even: " + even));
```

Output:
```
Even: 2
Even: 4
Even: 6
Even: 8
Even: 10
```

A realistic example: only letting through orders above a certain value.

```java
orderFlux
    .filter(order -> order.getTotal() > 100)
    .subscribe(order -> System.out.println("High value order: " + order.getId()));
```

## Why It Matters

You'll use `.filter()` constantly — anytime you want to narrow a stream down
to just the items you care about, without writing a manual `if` check inside
every operator that comes after it.
