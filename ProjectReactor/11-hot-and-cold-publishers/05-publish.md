# publish()

## In Simple Terms

`.publish()` converts a cold `Flux` into a `ConnectableFlux` — a special hot variant
that **won't start producing data until you explicitly call `.connect()`**. This
gives you precise control over exactly when the shared execution begins, regardless
of how many subscribers have already registered interest.

## Simple Example

```java
ConnectableFlux<Integer> published = Flux.range(1, 5)
    .doOnSubscribe(s -> System.out.println("Someone subscribed"))
    .publish();

published.subscribe(n -> System.out.println("Subscriber A: " + n));
published.subscribe(n -> System.out.println("Subscriber B: " + n));

System.out.println("Both subscribed, but nothing has run yet!");

published.connect(); // NOW the source actually starts, both subscribers get all items
```

Output:
```
Both subscribed, but nothing has run yet!
Someone subscribed
Subscriber A: 1
Subscriber B: 1
Subscriber A: 2
Subscriber B: 2
...
```

## Why It Matters

`.publish()` is useful when you need to **coordinate multiple subscribers** to all
start receiving data at exactly the same moment — e.g., waiting until all expected
consumers have registered before "kicking off" a shared, synchronized broadcast,
rather than starting as soon as the first subscriber arrives (which is what
`.share()` does automatically).
