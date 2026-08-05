# combineLatest()

## In Simple Terms

`Flux.combineLatest(source1, source2, combiner)` combines multiple publishers by
always pairing each new item with the **most recent (latest)** item from the other
source(s) — unlike `zip()`, which strictly pairs items 1-to-1 in order. It emits a
new combined value every time **any** source produces a new item (once all sources
have emitted at least once).

## Simple Example

```java
Flux<String> temperature = Flux.just("20C", "22C", "25C").delayElements(Duration.ofMillis(100));
Flux<String> humidity = Flux.just("40%", "45%").delayElements(Duration.ofMillis(150));

Flux.combineLatest(temperature, humidity, (temp, hum) -> temp + " / " + hum)
    .subscribe(System.out::println);
```

Possible output (exact interleaving depends on timing):
```
22C / 40%
22C / 45%
25C / 45%
```

Notice it doesn't wait for a matching pair like `zip()` would — it recombines
whenever *either* side updates, using the latest known value from the other side.

## zip() vs combineLatest()

| Aspect            | zip()                              | combineLatest()                         |
|--------------------|--------------------------------------|--------------------------------------------|
| Pairing            | Strict 1-to-1, in order               | Latest-known value from each source        |
| Emission trigger   | Only when ALL sources have a new item | Whenever ANY source emits a new item        |
| Use case           | Combining independent, parallel results | Reacting to live, continuously-updating sources |

## Why It Matters

`.combineLatest()` is ideal for **live dashboards** — e.g., always showing the most
recent stock price combined with the most recent exchange rate, recalculating
whenever either value updates, rather than waiting for both to tick in lockstep.
