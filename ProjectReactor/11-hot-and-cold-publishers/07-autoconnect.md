# autoConnect()

## In Simple Terms

`.autoConnect(minSubscribers)` (also used on a `ConnectableFlux`)
automatically starts the source once at least `minSubscribers` have
registered — but unlike `.refCount()`, it never shuts things down again
once subscribers start leaving. Once it's running, it just keeps running.

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

`.autoConnect()` is a good fit for expensive, long-lived shared resources
that should start once there are enough consumers ready, but shouldn't be
torn down and restarted just because the subscriber count briefly dips to
zero — like a shared WebSocket connection meant to stay open for the whole
life of the application once it's up and running.
