# Implementing a Custom Publisher

## In Simple Terms

Before relying on `Mono`/`Flux`, it's worth building a `Publisher` completely by
hand, using nothing but the raw Reactive Streams interfaces. This strips away all
of Reactor's convenience and shows you exactly what a publisher actually has to
do: respond to `subscribe()` by handing over a `Subscription`, and only send items
when `request(n)` is called.

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

Writing this by hand shows exactly how much careful bookkeeping — tracking
`remaining`, tracking `cancelled` — goes into being a correct publisher. That's
exactly why `Flux.range(1, 5)`, a single line, is so valuable: it handles all of
this correctly for you, every time.
