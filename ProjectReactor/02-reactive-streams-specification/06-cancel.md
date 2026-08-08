# cancel()

## In Simple Terms

`cancel()` is how a subscriber tells the publisher: **"Stop, I don't want any more
data."** After this is called, the publisher should stop sending items and clean
up anything tied to that subscriber.

```java
public interface Subscription {
    void request(long n);
    void cancel(); // <-- this one
}
```

## Simple Example

```java
Disposable subscription = Flux.interval(Duration.ofSeconds(1))
    .subscribe(tick -> System.out.println("Tick: " + tick));

// later, stop receiving further ticks:
subscription.dispose(); // internally calls cancel() on the underlying Subscription
```

In everyday Project Reactor code, you almost never call `cancel()` yourself —
`.subscribe()` gives you back a `Disposable`, and calling `.dispose()` on that
does the cancelling for you.

## Why It Matters

Cancellation is how resources get cleaned up. Picture an endless stream (like
`Flux.interval()`) hooked up to a browser tab. The moment the user closes that
tab, the connection drops, and Spring WebFlux automatically cancels the stream
behind it — so the server stops doing pointless work for a client that's already
gone.
