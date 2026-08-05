# Cold Publisher

## In Simple Terms

A **cold publisher** starts its data production **fresh, from the beginning, for
every new subscriber**. Two subscribers never share the same "run" — each gets its
own independent execution, like each viewer starting a video-on-demand movie from
frame zero.

## Simple Example

```java
Flux<Long> cold = Flux.just(System.currentTimeMillis());

cold.subscribe(t -> System.out.println("Subscriber 1 got: " + t));

try { Thread.sleep(1000); } catch (Exception e) {}

cold.subscribe(t -> System.out.println("Subscriber 2 got: " + t));
```

Output (different timestamps — each subscription re-executed the source):
```
Subscriber 1 got: 1732000000000
Subscriber 2 got: 1732000001000
```

Most sources you create directly (`Flux.just()`, `Flux.fromIterable()`, a database
query, an HTTP call) are cold by default.

## Why It Matters

Cold is usually the **correct default** — e.g., a `Mono<User>` representing a
database lookup should re-run per subscriber to fetch fresh data, not share one
cached execution across unrelated callers. When you genuinely want subscribers to
share a single, ongoing execution (like a live broadcast), you deliberately convert
a cold publisher into a hot one using operators like `.share()` or `.publish()`.
