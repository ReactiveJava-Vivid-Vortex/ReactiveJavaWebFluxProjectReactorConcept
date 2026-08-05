# collectMap()

## In Simple Terms

`.collectMap(keyExtractor, valueExtractor)` gathers all items from a `Flux` into a
single `Mono<Map<K, V>>`, using functions you supply to determine each item's key and
value. It's the reactive equivalent of `Collectors.toMap()` in the Java Streams API.

## Simple Example

```java
Flux.just(
    new Order("O1", 100),
    new Order("O2", 250),
    new Order("O3", 75)
)
.collectMap(Order::getId, Order::getTotal)
.subscribe(map -> System.out.println("Order totals: " + map));
```

Output:
```
Order totals: {O1=100, O2=250, O3=75}
```

If you only pass a key extractor, the whole item becomes the value:

```java
Flux.just("apple", "banana", "cherry")
    .collectMap(String::length) // keys: word length, values: the word itself
    .subscribe(map -> System.out.println(map));
// {5=apple, 6=cherry, ...} - note: only the LAST item per duplicate key is kept!
```

**Important gotcha:** if two items produce the same key, the later one **overwrites**
the earlier one in the resulting map — duplicates are silently lost, not combined.

## Why It Matters

`.collectMap()` is handy for quickly building lookup tables from a stream of data
(e.g., "map of product ID to product") for fast in-memory access later in your
pipeline, without manually looping and populating a `HashMap` yourself.
