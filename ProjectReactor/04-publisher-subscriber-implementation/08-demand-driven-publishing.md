# Demand-Driven Publishing

## In Simple Terms

"Demand-driven publishing" means a publisher only makes an item when there's
actual demand for it — that is, only in response to `request(n)`. It never
eagerly generates a bunch of stuff upfront, hoping someone will want it later.

## Simple Example

```java
Flux<Integer> demandDriven = Flux.generate(sink -> {
    System.out.println("Generating a value..."); // only runs when demand exists
    sink.next((int) (Math.random() * 100));
});

demandDriven
    .take(3) // only request 3 items total
    .subscribe(value -> System.out.println("Got: " + value));
```

Notice "Generating a value..." prints **exactly 3 times** — matching the demand
from `.take(3)`, no more, no less:

```
Generating a value...
Got: 42
Generating a value...
Got: 17
Generating a value...
Got: 89
```

## Why It Matters

This is what keeps reactive streams safe with huge or endless sources — like a
massive file or a never-ending sensor feed. Since new items are only made when
there's demand for them, you never end up with a pile of unused data building up
in memory. The source naturally slows down to match whatever's consuming it.
