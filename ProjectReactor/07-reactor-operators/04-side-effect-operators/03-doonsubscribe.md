# doOnSubscribe()

## In Simple Terms

`.doOnSubscribe(consumer)` runs a side effect exactly when a new subscription is
established at this point in the pipeline — useful for logging "the stream just
started" or capturing a start timestamp for measuring duration.

## Simple Example

```java
Mono.just("data")
    .doOnSubscribe(subscription -> System.out.println("Subscribed at: " + Instant.now()))
    .subscribe(value -> System.out.println("Value: " + value));
```

Combining with `.doFinally()` to measure elapsed time:

```java
long start = System.currentTimeMillis();

Mono.delay(Duration.ofMillis(500))
    .doOnSubscribe(s -> System.out.println("Started timing"))
    .doFinally(signal -> {
        long elapsed = System.currentTimeMillis() - start;
        System.out.println("Took " + elapsed + "ms, ended with: " + signal);
    })
    .subscribe();
```

## Why It Matters

`.doOnSubscribe()` is a natural hook point for setup logic tied specifically to
subscription time — for instance, incrementing an "active requests" gauge metric
when a request pipeline begins, later decremented in `.doFinally()`.
