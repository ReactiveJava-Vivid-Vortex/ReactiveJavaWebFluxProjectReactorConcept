# request(n)

## In Simple Terms

`request(n)` is how a `Subscriber` tells its `Subscription` **"I am ready for `n` more
items."** This is the mechanism through which backpressure is implemented — the
publisher can never send more items than have been requested in total.

## Simple Example

```java
subscriber.onSubscribe(subscription -> {
    subscription.request(3); // "send me exactly 3 items"
});

// publisher sends onNext() 3 times, then must WAIT
// until request() is called again before sending more
```

A common pattern is requesting `Long.MAX_VALUE`, which effectively means "send
everything as fast as you can, I won't apply backpressure":

```java
subscription.request(Long.MAX_VALUE); // unlimited/unbounded request
```

## Why It Matters

`request(n)` is the single most important mechanism in the entire Reactive Streams
spec. Without it, publishers could overwhelm subscribers with data faster than they
can process it — leading to `OutOfMemoryError`s. In Project Reactor, operators like
`.limitRate(n)` let you control the request size flowing through a pipeline, which is
extremely useful when working with slow consumers (e.g., writing to a rate-limited
external API).
