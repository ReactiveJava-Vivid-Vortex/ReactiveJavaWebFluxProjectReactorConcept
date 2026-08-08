# doOnSubscribe()

## In Simple Terms

`.doOnSubscribe()` runs something the moment a subscription starts at this
point in the pipeline — good for logging "we just started" or grabbing a
timestamp so you can measure how long things take.

## Simple Example

```java
Mono.just("data")
    .doOnSubscribe(subscription -> System.out.println("Subscribed at: " + Instant.now()))
    .subscribe(value -> System.out.println("Value: " + value));
```

Pairing it with `.doFinally()` to measure how long something took:

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

`.doOnSubscribe()` is a natural spot for setup work tied to "the stream just
began" — for example, bumping up an "active requests" counter when a
request starts, which you'd then bring back down in `.doFinally()`.
