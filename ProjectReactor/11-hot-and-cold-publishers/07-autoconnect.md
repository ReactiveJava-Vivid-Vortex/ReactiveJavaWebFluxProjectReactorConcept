# autoConnect()

## In Simple Terms

`.autoConnect(minSubscribers)` (also called on a `ConnectableFlux`) automatically
connects (starts the source) once at least `minSubscribers` subscribers have
registered — but, unlike `.refCount()`, it does **not** disconnect when subscribers
later leave. Once started, it keeps running regardless of how many subscribers
remain.

## Simple Example

```java
Flux<Long> autoStarted = Flux.interval(Duration.ofSeconds(1))
    .doOnSubscribe(s -> System.out.println("Subscribed"))
    .publish()
    .autoConnect(2); // waits for 2 subscribers before starting

autoStarted.subscribe(t -> System.out.println("A: " + t));
System.out.println("Only 1 subscriber so far - not started yet");

autoStarted.subscribe(t -> System.out.println("B: " + t));
System.out.println("2nd subscriber arrived - now it starts!");
```

## refCount() vs autoConnect()

| Aspect                          | refCount()                          | autoConnect()                          |
|----------------------------------|----------------------------------------|-------------------------------------------|
| Starts when...                   | First subscriber(s) arrive              | Minimum subscriber count reached           |
| Stops when...                    | Last subscriber leaves                  | Never stops automatically once started    |
| Typical use                      | On-demand shared resource                | "Warm up once, keep running" scenarios     |

## Why It Matters

`.autoConnect()` is useful for expensive, long-lived shared resources that should
start once enough consumers are ready, but shouldn't be torn down and restarted every
time subscriber counts briefly dip to zero — e.g., a shared WebSocket connection that
should stay open for the lifetime of the application after initial startup.
