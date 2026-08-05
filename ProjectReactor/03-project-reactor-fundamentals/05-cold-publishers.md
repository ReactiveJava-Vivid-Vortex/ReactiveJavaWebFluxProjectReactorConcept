# Cold Publishers

## In Simple Terms

A **cold publisher** re-runs its data-producing logic **from scratch, for every new
subscriber**. Each subscriber gets its own independent, fresh copy of the sequence —
like watching a movie on Netflix (video-on-demand): every viewer starts from the
beginning, whenever they press play.

Most `Mono`/`Flux` sources you create (`Flux.just()`, `Flux.fromIterable()`,
a database query, an HTTP call) are **cold** by default.

## Simple Example

```java
Flux<Long> coldFlux = Flux.just(System.currentTimeMillis());

coldFlux.subscribe(time -> System.out.println("Subscriber 1: " + time));

try { Thread.sleep(2000); } catch (InterruptedException e) {}

coldFlux.subscribe(time -> System.out.println("Subscriber 2: " + time));
```

Output (timestamps differ — each subscriber triggers a fresh execution):
```
Subscriber 1: 1732000000000
Subscriber 2: 1732000002000
```

Even though it's the *same* `coldFlux` object, subscribing twice re-executes the
`System.currentTimeMillis()` call separately for each subscriber.

## Why It Matters

Cold publishers are the default and usually what you want — e.g., a database query
should re-run per subscriber to get fresh data, not share one cached run. When you
*do* want subscribers to share the same, single execution (like one live broadcast
instead of one full replay per viewer), you convert a cold publisher into a **hot**
one, using operators like `.share()` or `.publish()` (covered in the "Hot & Cold
Publishers" section).
