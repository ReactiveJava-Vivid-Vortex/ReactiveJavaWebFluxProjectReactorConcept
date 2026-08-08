# firstWithSignal()

## In Simple Terms

`Flux.firstWithSignal()` starts several streams at once and just keeps
whichever one responds first — a value, an error, or completion, doesn't
matter which — and cancels all the rest. It's a straight-up race: first to
answer wins, everyone else gets called off.

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

`.firstWithSignal()` is a solid pattern for redundant or fallback calls —
querying two replica servers, or a primary and backup source, at the same
time and just using whichever answers first, calling off the slower one to
save resources. It's a common trick for cutting down worst-case latency in
distributed systems.

(This operator replaced the older, now-deprecated `firstEmitting()`/`first()`
methods in newer Reactor versions.)
