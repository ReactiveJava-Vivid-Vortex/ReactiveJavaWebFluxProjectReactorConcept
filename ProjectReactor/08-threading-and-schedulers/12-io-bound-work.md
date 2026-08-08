# I/O-Bound Work

## In Simple Terms

I/O-bound work spends most of its time just *waiting* — for a database to
respond, for another service to answer an HTTP call, for a file to load
from disk. The CPU is mostly sitting idle during that wait; the real
bottleneck is how long the network or disk takes, not how fast the
processor is.

## Simple Example

```java
// Non-blocking I/O-bound call - no scheduler switch needed, WebClient is already non-blocking
webClient.get()
    .uri("/api/users/{id}", userId)
    .retrieve()
    .bodyToMono(User.class)
    .subscribe(user -> System.out.println("Got user: " + user));

// Blocking I/O-bound call (legacy) - MUST be isolated on boundedElastic()
Mono.fromCallable(() -> legacyBlockingHttpCall(userId))
    .subscribeOn(Schedulers.boundedElastic())
    .subscribe(user -> System.out.println("Got user: " + user));
```

## Why It Matters

This is exactly the kind of work reactive programming and non-blocking I/O
were built to handle well. A truly non-blocking call (like Spring's
`WebClient`) doesn't hog a thread while it waits — the thread gets freed up
immediately and reused elsewhere, with a callback picking things back up
once data actually arrives. Old-style *blocking* calls, though, need to be
deliberately fenced off (with `subscribeOn(Schedulers.boundedElastic())`) so
they don't clog the small handful of threads meant for non-blocking work.
