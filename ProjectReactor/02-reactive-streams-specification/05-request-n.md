# request(n)

## In Simple Terms

`request(n)` is how a subscriber tells its `Subscription`, **"I'm ready for `n`
more items."** This one call is the entire trick behind backpressure — a
publisher is never allowed to send more than what's been asked for.

## Simple Example

```java
subscriber.onSubscribe(subscription -> {
    subscription.request(3); // "send me exactly 3 items"
});

// publisher sends onNext() 3 times, then must WAIT
// until request() is called again before sending more
```

A common shortcut is asking for `Long.MAX_VALUE`, which basically means "just
send everything, don't hold back":

```java
subscription.request(Long.MAX_VALUE); // unlimited/unbounded request
```

## Why It Matters

`request(n)` is arguably the single most important idea in the whole spec.
Without it, a publisher could send data faster than a subscriber can handle,
eventually crashing the app with an `OutOfMemoryError`. In Project Reactor,
operators like `.limitRate(n)` are really just a friendlier way of controlling
this same request size — handy when talking to something slow, like a
rate-limited external API.
