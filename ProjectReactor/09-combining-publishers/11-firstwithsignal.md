# firstWithSignal()

## In Simple Terms

`Flux.firstWithSignal(source1, source2, ...)` subscribes to multiple publishers at
once and takes **only the first one to emit any signal** (a value, an error, or
completion) — the "winner" — and cancels all the others. It's a race between
sources; whichever responds first wins entirely.

## Simple Example

```java
Mono<String> serverA = callServer("A").delayElement(Duration.ofMillis(200));
Mono<String> serverB = callServer("B").delayElement(Duration.ofMillis(100));

Mono.firstWithSignal(serverA, serverB)
    .subscribe(result -> System.out.println("Winner: " + result));
```

Output (Server B responds faster, so it wins, and Server A's call is cancelled):
```
Winner: Response from B
```

## Why It Matters

`.firstWithSignal()` is a powerful pattern for **redundant/fallback calls** — e.g.,
querying two replica servers or a primary/backup data source simultaneously, and
using whichever responds first, cancelling the slower one to save resources. This is
a common resilience pattern for reducing tail latency in distributed systems.

(Note: this operator replaced the older, now-deprecated `firstEmitting()`/`first()`
methods in newer Reactor versions.)
