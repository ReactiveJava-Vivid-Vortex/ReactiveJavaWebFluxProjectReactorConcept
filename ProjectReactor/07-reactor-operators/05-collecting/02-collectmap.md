# collectMap()

## In Simple Terms

`.collectMap()` gathers everything from a `Flux` into a single lookup table
(a `Map`), using rules you provide for what becomes the key and what becomes
the value for each item. It's the reactive version of building a `HashMap`
by hand in a loop, just done for you.

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

If you only give it a way to build the key, it just uses the whole item as
the value:

```java
Flux.just("apple", "banana", "cherry")
    .collectMap(String::length) // keys: word length, values: the word itself
    .subscribe(map -> System.out.println(map));
// {5=apple, 6=cherry, ...} - note: only the LAST item per duplicate key is kept!
```

**Watch out for this:** if two items end up with the same key, the newer one
quietly replaces the older one in the map — nothing gets combined, one of
them just disappears.

## Why It Matters

`.collectMap()` is great for quickly turning a stream of data into a
fast lookup table (like "product ID → product") that you can query
instantly later, without writing the loop-and-put logic yourself.
