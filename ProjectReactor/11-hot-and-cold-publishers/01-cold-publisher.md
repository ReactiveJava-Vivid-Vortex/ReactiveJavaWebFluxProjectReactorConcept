# Cold Publisher

## In Simple Terms

A cold publisher starts fresh, from scratch, for every single subscriber —
nobody shares a run with anybody else. It's like watching a movie on a
streaming service: every viewer starts from frame one, no matter when they
hit play.

## Simple Example

```java
Flux<Long> cold = Flux.just(System.currentTimeMillis());

cold.subscribe(t -> System.out.println("Subscriber 1 got: " + t));

try { Thread.sleep(1000); } catch (Exception e) {}

cold.subscribe(t -> System.out.println("Subscriber 2 got: " + t));
```

Output (different timestamps — each subscription re-ran the source from
scratch):
```
Subscriber 1 got: 1732000000000
Subscriber 2 got: 1732000001000
```

Most sources you build yourself — `Flux.just()`, `Flux.fromIterable()`, a
database query, an HTTP call — are cold by default.

## Why It Matters

Cold is usually exactly what you want — a `Mono<User>` from a database
lookup should run fresh for each subscriber and grab up-to-date data, not
hand out one shared, possibly stale result to everyone who asks. When you
genuinely want subscribers to share one ongoing run — like tuning into a
live broadcast — you deliberately turn a cold publisher into a hot one
using operators like `.share()` or `.publish()`.
