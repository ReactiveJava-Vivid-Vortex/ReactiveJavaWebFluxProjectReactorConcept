# Implementing a Custom Subscriber

## In Simple Terms

Writing your own `Subscriber` lets you control exactly how much data you ask for
and when, by implementing all four callback methods yourself. It's more work than
Reactor's `.subscribe(consumer)` shortcut, but it makes it very clear how demand
and data actually connect.

## Simple Example

```java
import org.reactivestreams.*;

public class LoggingSubscriber implements Subscriber<Integer> {
    private Subscription subscription;

    @Override
    public void onSubscribe(Subscription s) {
        this.subscription = s;
        System.out.println("Subscribed! Requesting 2 items.");
        s.request(2); // only ask for 2 to start
    }

    @Override
    public void onNext(Integer item) {
        System.out.println("Received: " + item);
        // request one more each time we process one, keeping demand at ~2
        subscription.request(1);
    }

    @Override
    public void onError(Throwable t) {
        System.out.println("Error: " + t.getMessage());
    }

    @Override
    public void onComplete() {
        System.out.println("Stream completed.");
    }
}

// Usage:
Flux.range(1, 5).subscribe(new LoggingSubscriber());
```

## Why It Matters

Reactor gives you `BaseSubscriber<T>` so you almost never have to implement the
raw `Subscriber` interface yourself. It comes with sensible defaults and handy
hooks (`hookOnSubscribe`, `hookOnNext`, etc.), while still letting you control
demand by hand when you need to — say, to slow things down for a struggling
downstream consumer.
