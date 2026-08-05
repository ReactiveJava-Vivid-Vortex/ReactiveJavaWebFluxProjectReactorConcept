# boundedElastic()

## In Simple Terms

`Schedulers.boundedElastic()` provides a thread pool designed specifically for
**blocking** or slow I/O work that you can't avoid (e.g., a legacy blocking JDBC
call, file I/O). It grows its thread pool dynamically as needed (up to a large but
bounded cap, e.g., 10x CPU cores), reusing idle threads when possible.

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

`boundedElastic()` exists specifically so that unavoidable blocking calls don't tie
up the small, precious pool of event-loop threads (used by `parallel()` / Netty)
that are meant to handle many concurrent non-blocking requests. Running a blocking
JDBC call directly on an event-loop thread can stall many unrelated requests; wrapping
it with `.subscribeOn(Schedulers.boundedElastic())` isolates the damage to a
dedicated pool built to absorb blocking work.

**Rule of thumb:** any call you can't make non-blocking (legacy libraries, blocking
JDBC, `Thread.sleep()`) should run on `boundedElastic()`, never on `parallel()` or an
event-loop thread.
