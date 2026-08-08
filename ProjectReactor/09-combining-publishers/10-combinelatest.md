# combineLatest()

## In Simple Terms

`Flux.combineLatest()` pairs each new item with whatever the *most recent*
item from the other stream happened to be — not a strict 1-to-1 match like
`zip()`. Every time either side updates, it recombines using the newest
value it has from each side. Think of a weather app showing temperature and
humidity together — every time either reading updates, the display
refreshes using the latest of both, not waiting for them to update in sync.

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

Notice it doesn't wait for a matching pair the way `zip()` would — it
recombines whenever *either* side updates, always using the latest known
value from the other side.

## zip() vs combineLatest()

| Aspect            | zip()                              | combineLatest()                         |
|--------------------|--------------------------------------|--------------------------------------------|
| Pairing            | Strict 1-to-1, in order               | Latest-known value from each source        |
| Emission trigger   | Only when ALL sources have a new item | Whenever ANY source emits a new item        |
| Use case           | Combining independent, parallel results | Reacting to live, continuously-updating sources |

## Why It Matters

`.combineLatest()` is perfect for live dashboards — always showing the
freshest stock price alongside the freshest exchange rate, recalculating
the moment either value changes, instead of waiting for both to tick in
lockstep.
