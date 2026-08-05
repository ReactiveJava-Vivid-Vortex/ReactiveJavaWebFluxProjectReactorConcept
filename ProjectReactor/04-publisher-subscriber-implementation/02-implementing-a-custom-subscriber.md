# Implementing a Custom Subscriber

## In Simple Terms

A custom `Subscriber` lets you precisely control how much data you request and when,
by implementing all four callback methods yourself. This is more manual than using
Reactor's `.subscribe(consumer)` shortcut, but it shows you exactly how demand and
data flow are connected.

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

In Project Reactor, `BaseSubscriber<T>` is provided specifically so you don't need to
implement the raw `Subscriber` interface yourself — it gives you sensible defaults and
hooks (`hookOnSubscribe`, `hookOnNext`, etc.) while still letting you control demand
manually when needed (e.g., for fine-grained backpressure in a slow consumer).
