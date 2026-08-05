# collectList()

## In Simple Terms

`.collectList()` gathers all items from a `Flux` into a single `Mono<List<T>>`. It
waits for the source to complete, then emits the whole collected list as one value.
This is the reactive way of turning "many items over time" back into "one regular
Java `List`."

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

A common real-world use: returning a full list from a reactive repository as a single
response.

```java
Mono<List<Order>> allOrders = orderRepository.findAll().collectList();
```

**Important:** `.collectList()` requires the source `Flux` to eventually complete —
using it on an infinite stream (like `Flux.interval()`) will simply never emit,
since it can't know the list is "final" until `onComplete()` fires. It also loads
everything into memory at once, so it's not appropriate for very large or unbounded
streams.

## Why It Matters

`.collectList()` is the natural bridge back from streaming to a regular collection —
useful whenever a consumer genuinely needs the full data set at once (e.g., to sort
it, or to serialize the whole thing as one JSON array).
