# Common Misconceptions About Reactive Programming

## In Simple Terms

Reactive programming is one of the most misunderstood topics in Java. Here are the
most common myths, and the reality behind each one.

### Myth 1: "Reactive is always faster"

**Reality:** Reactive doesn't make individual operations faster. A single reactive
call to a database isn't quicker than a blocking one — the network/DB latency is the
same either way. What reactive improves is **throughput under high concurrency**,
because threads aren't wasted on waiting.

### Myth 2: "Reactive automatically runs on multiple threads / makes things parallel"

**Reality:** By default, a reactive pipeline runs on **whatever thread called
`subscribe()`**, sequentially — it isn't automatically parallel. You must explicitly
use operators like `subscribeOn()` / `parallel()` if you want to shift work onto
other threads.

```java
Mono.just("hello")
    .map(String::toUpperCase) // runs on the SAME thread that subscribed, by default
    .subscribe(System.out::println);
```

### Myth 3: "If I mix in one blocking call, it's fine, it's just one line"

**Reality:** A single blocking call (e.g., a JDBC query) inside a reactive pipeline
can freeze one of your few precious event-loop threads, stalling *many* unrelated
requests that happen to share that thread. This is one of the most dangerous mistakes
in reactive code.

### Myth 4: "Reactive programming replaces the need for good design"

**Reality:** Reactive doesn't fix bad architecture. If your database is slow or your
downstream service is unreliable, reactive code will still be waiting on those things
— it just won't waste a thread while doing so.

### Myth 5: "You need reactive everywhere for it to help"

**Reality:** You don't have to reactive-ify your entire stack overnight. Reactive
value shows up most where you have high concurrency and I/O waiting; other parts of a
system can remain traditional if that's simpler and sufficient.

## Why It Matters

Believing these myths leads to two common mistakes: expecting reactive code to be a
free performance upgrade, or accidentally blocking inside a reactive pipeline and
wondering why the whole application seems to freeze under load. Understanding what
reactive **actually does** (non-blocking scheduling, not automatic speed or
parallelism) avoids both traps.
