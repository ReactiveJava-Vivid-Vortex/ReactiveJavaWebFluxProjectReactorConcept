# boundedElastic()

## In Simple Terms

`Schedulers.boundedElastic()` is a thread pool made specifically for slow
or blocking work you can't avoid — an old-style blocking database call, or
reading a file. It grows as needed (up to a fairly generous cap, like 10x
your CPU count), reusing threads when it can, kind of like a rideshare
company adding more drivers when demand spikes.

## Simple Example

```java
Mono.fromCallable(() -> {
    // simulate a blocking call, like legacy JDBC
    Thread.sleep(1000);
    return "Blocking result";
})
.subscribeOn(Schedulers.boundedElastic())
.subscribe(result -> System.out.println("Got: " + result + " on " + Thread.currentThread().getName()));
```

Output (thread name will be something like `boundedElastic-1`):
```
Got: Blocking result on boundedElastic-1
```

## Why It Matters

`boundedElastic()` exists so blocking calls don't clog up the small,
precious set of threads meant to juggle lots of concurrent, non-blocking
requests. Run a blocking database call directly on one of those precious
threads, and it can stall a bunch of unrelated requests at once. Move it to
`boundedElastic()` instead, and the damage stays contained to a pool built
to soak it up.

**Rule of thumb:** anything you can't make non-blocking — old libraries,
blocking JDBC, `Thread.sleep()` — belongs on `boundedElastic()`, never on
`parallel()` or an event-loop thread.
