# Demand-Driven Publishing

## In Simple Terms

"Demand-driven publishing" means a publisher only produces items **when there is
demand for them** (i.e., only in response to `request(n)`), rather than eagerly
generating everything upfront regardless of whether anyone is ready to consume it.
This is the opposite of, say, eagerly filling a `List` and handing it over all at
once.

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

Output shows "Generating a value..." only printing **3 times** — exactly matching the
demand from `.take(3)`, never more:

```
Generating a value...
Got: 42
Generating a value...
Got: 17
Generating a value...
Got: 89
```

## Why It Matters

Demand-driven publishing is what makes reactive streams memory-safe when working with
huge or infinite data sources (e.g., reading a massive file, or an endless sensor
feed). Because production only happens in response to demand, you never build up an
unbounded backlog of unconsumed items in memory — the producer naturally paces itself
to match the consumer.
