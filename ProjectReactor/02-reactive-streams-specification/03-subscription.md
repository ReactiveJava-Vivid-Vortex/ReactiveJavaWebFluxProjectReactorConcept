# Subscription

## In Simple Terms

A `Subscription` is like a **remote control** handed to the subscriber the moment
it subscribes. With it, the subscriber can ask for more items (`request(n)`) or
tell the publisher to stop entirely (`cancel()`).

```java
public interface Subscription {
    void request(long n);
    void cancel();
}
```

## Simple Example

Think of it like ordering food in courses at a restaurant, instead of getting
everything dumped on the table at once:

```java
public void onSubscribe(Subscription subscription) {
    // "Please send me 2 items to start"
    subscription.request(2);

    // later, if we've had enough:
    // subscription.cancel();
}
```

The publisher is **not allowed** to send more than what's been asked for. If the
subscriber only calls `request(2)`, the publisher can send at most 2 items before
it has to wait for another request.

## Why It Matters

The `Subscription` is what makes backpressure possible in the first place. Without
it, a fast publisher could flood a slow subscriber with more data than it can
handle, and memory would blow up. With it, the subscriber always stays in control
of the pace.
