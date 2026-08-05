# Subscription

## In Simple Terms

A `Subscription` is the **handshake object** created when a `Subscriber` subscribes to
a `Publisher`. It's the "remote control" the subscriber uses to control the flow of
data — asking for more items (`request(n)`) or stopping the stream entirely
(`cancel()`).

```java
public interface Subscription {
    void request(long n);
    void cancel();
}
```

## Simple Example

Think of a subscription like ordering food in installments at a restaurant, one plate
at a time:

```java
public void onSubscribe(Subscription subscription) {
    // "Please send me 2 items to start"
    subscription.request(2);

    // later, if we've had enough:
    // subscription.cancel();
}
```

The publisher is **not allowed** to send more items than have been requested. If the
subscriber only calls `request(2)`, the publisher must send at most 2 `onNext()`
signals until more are requested.

## Why It Matters

The `Subscription` is what makes **backpressure** possible — the whole point of the
Reactive Streams spec. Without it, a fast publisher could flood a slow subscriber
with more data than it can handle, causing memory overload. With it, the consumer
stays in control of the pace.
