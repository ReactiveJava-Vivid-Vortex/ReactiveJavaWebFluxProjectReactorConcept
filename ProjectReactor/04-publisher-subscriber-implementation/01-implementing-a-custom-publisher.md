# Implementing a Custom Publisher

## In Simple Terms

Before using `Mono`/`Flux`, it helps to build a `Publisher` completely from scratch,
by hand, using only the raw Reactive Streams interfaces. This strips away all the
convenience methods Reactor gives you and shows exactly what a publisher has to do:
respond to `subscribe()` by handing the subscriber a `Subscription`, and only emit
items when `request(n)` is called.

## Simple Example

```java
import org.reactivestreams.*;

public class RangePublisher implements Publisher<Integer> {
    private final int start;
    private final int count;

    public RangePublisher(int start, int count) {
        this.start = start;
        this.count = count;
    }

    @Override
    public void subscribe(Subscriber<? super Integer> subscriber) {
        subscriber.onSubscribe(new Subscription() {
            int current = start;
            int remaining = count;
            boolean cancelled = false;

            @Override
            public void request(long n) {
                for (long i = 0; i < n && remaining > 0 && !cancelled; i++) {
                    subscriber.onNext(current++);
                    remaining--;
                }
                if (remaining == 0 && !cancelled) {
                    subscriber.onComplete();
                }
            }

            @Override
            public void cancel() {
                cancelled = true;
            }
        });
    }
}

// Usage:
new RangePublisher(1, 5).subscribe(new Subscriber<>() {
    public void onSubscribe(Subscription s) { s.request(Long.MAX_VALUE); }
    public void onNext(Integer item) { System.out.println("Got: " + item); }
    public void onError(Throwable t) { }
    public void onComplete() { System.out.println("Done"); }
});
```

## Why It Matters

Writing this by hand reveals just how much boilerplate and careful bookkeeping (like
tracking `remaining` and `cancelled`) is required to be a correct, spec-compliant
publisher. This is exactly why Project Reactor's `Flux.range(1, 5)` — a single line —
is so valuable: it handles all of this correctly and safely for you.
