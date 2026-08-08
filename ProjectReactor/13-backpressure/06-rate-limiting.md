# Rate Limiting

## In Simple Terms

Rate limiting means deliberately slowing down how fast items move through a
pipeline — often not because your app can't keep up, but because something
outside your control (like a third-party API's requests-per-second cap)
demands it.

## Simple Example

Using `.delayElements()` to space things out:

```java
Flux.range(1, 10)
    .delayElements(Duration.ofMillis(200)) // emit at most one every 200ms
    .subscribe(n -> System.out.println("Calling API with: " + n));
```

Using `.limitRate(n)` to pull from the source in smaller chunks, instead of
all at once (handy for huge or expensive sources):

```java
Flux.range(1, 1_000_000)
    .limitRate(50) // request from the source in batches of 50, not all at once
    .subscribe(n -> processItem(n));
```

Combining rate limiting with a cap on how many things run at once (say,
only 5 outgoing calls in flight at a time):

```java
Flux.fromIterable(userIds)
    .flatMap(id -> callExternalApi(id), 5) // max concurrency of 5
    .subscribe(response -> System.out.println("Got: " + response));
```

## Why It Matters

Rate limiting matters whenever your pipeline talks to something with its
own limits — API quotas, a database's connection pool. Without it, your
reactive code could technically fire off requests faster than the outside
world can safely handle, leading to `429 Too Many Requests` errors or
knocking over a downstream system.
