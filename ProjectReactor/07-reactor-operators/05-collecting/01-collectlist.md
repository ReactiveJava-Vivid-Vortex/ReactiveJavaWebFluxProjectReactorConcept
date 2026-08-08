# collectList()

## In Simple Terms

`.collectList()` waits for a `Flux` to finish, scoops up everything it
produced along the way, and hands it all back as one regular Java `List`,
wrapped in a `Mono`. It's how you turn "a bunch of items arriving over time"
back into an ordinary list you already know how to work with.

## Simple Example

```java
Flux.just(1, 2, 3, 4, 5)
    .collectList()
    .subscribe(list -> System.out.println("Collected: " + list));
```

Output:
```
Collected: [1, 2, 3, 4, 5]
```

A common real-world use: returning a full list from a reactive repository as
a single response.

```java
Mono<List<Order>> allOrders = orderRepository.findAll().collectList();
```

**Good to know:** `.collectList()` needs the source `Flux` to actually
finish before it can hand you anything — try it on something endless (like
`Flux.interval()`) and it will just sit there forever, since it never gets
the "that's everything" signal. It also has to hold every item in memory at
once, so it's not a great fit for huge or unbounded streams.

## Why It Matters

`.collectList()` is the natural bridge back from "streaming" to "a regular
list" — useful whenever you genuinely need the whole set of data at once, to
sort it or send it back as one JSON array, for example.
