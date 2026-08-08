# Subscription (Custom Implementation)

## In Simple Terms

When you write your own `Publisher`, you also have to write your own
`Subscription` — the object that keeps track of how much has been requested and
actually sends items when `request(n)` is called. This is the trickiest part to
get right, because it has to safely deal with `request()` and `cancel()` being
called at any time, even from different threads.

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

A correctly built `Subscription` guarantees two things: the subscriber is never
sent more than it asked for, and cancelling actually stops things promptly.
Getting either wrong — like ignoring `cancel()`, or losing track of how much
was requested — causes exactly the kind of subtle bugs that are painful to track
down. That's why almost nobody writes this by hand in real projects and just
trusts Reactor's tested implementations instead.
