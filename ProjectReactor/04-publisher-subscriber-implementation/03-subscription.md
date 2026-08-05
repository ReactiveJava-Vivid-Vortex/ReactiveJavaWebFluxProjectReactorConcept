# Subscription (Custom Implementation)

## In Simple Terms

When implementing a `Publisher` from scratch, you must also implement your own
`Subscription` object — the thing responsible for tracking outstanding demand and
actually pushing items to the subscriber when `request(n)` is called. This is the
trickiest part to get right, since it must safely handle concurrent calls to
`request()` and `cancel()`.

## Simple Example

```java
class SimpleSubscription implements Subscription {
    private final Subscriber<? super Integer> subscriber;
    private final int[] data;
    private int index = 0;
    private volatile boolean cancelled = false;

    SimpleSubscription(Subscriber<? super Integer> subscriber, int[] data) {
        this.subscriber = subscriber;
        this.data = data;
    }

    @Override
    public void request(long n) {
        if (cancelled) return;
        for (int i = 0; i < n && index < data.length; i++) {
            subscriber.onNext(data[index++]);
        }
        if (index == data.length && !cancelled) {
            subscriber.onComplete();
        }
    }

    @Override
    public void cancel() {
        cancelled = true;
    }
}
```

## Why It Matters

A correctly implemented `Subscription` guarantees that a subscriber is never sent more
items than it asked for, and that cancellation stops emissions promptly. Getting this
logic wrong (e.g., ignoring `cancel()`, or not tracking demand precisely) is a classic
source of subtle reactive bugs — which is exactly why almost nobody hand-writes this
in real projects and instead relies on Project Reactor's tested implementations.
