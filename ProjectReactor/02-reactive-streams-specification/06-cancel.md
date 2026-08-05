# cancel()

## In Simple Terms

`cancel()` is how a `Subscriber` tells the `Publisher`: **"Stop sending me data, I'm
no longer interested."** After calling `cancel()`, the publisher should stop emitting
`onNext()` signals to that subscriber and release any resources tied to that
subscription.

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

In Project Reactor, you rarely call `Subscription.cancel()` directly — instead, the
`subscribe()` call returns a `Disposable`, and calling `.dispose()` on it triggers
cancellation under the hood.

## Why It Matters

Cancellation is essential for **resource cleanup**. Imagine an infinite stream (like
`Flux.interval(...)`) tied to a browser connection — when the user closes their
browser tab, the HTTP connection closes, and Spring WebFlux automatically cancels the
underlying subscription so the server stops doing unnecessary work for a client that's
no longer listening.
