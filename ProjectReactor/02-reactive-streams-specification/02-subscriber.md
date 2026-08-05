# Subscriber

## In Simple Terms

A `Subscriber` is the **consumer** of the data a `Publisher` produces. It's like a
viewer tuning in to a TV channel. It reacts to four possible signals from the
publisher: a subscription being set up, a new item, an error, or completion.

```java
public interface Subscriber<T> {
    void onSubscribe(Subscription s);
    void onNext(T t);
    void onError(Throwable t);
    void onComplete();
}
```

## Simple Example

```java
Subscriber<String> subscriber = new Subscriber<>() {
    private Subscription subscription;

    public void onSubscribe(Subscription s) {
        this.subscription = s;
        s.request(1); // ask for just 1 item to start
    }

    public void onNext(String item) {
        System.out.println("Received: " + item);
        subscription.request(1); // ask for the next one
    }

    public void onError(Throwable t) {
        System.out.println("Something went wrong: " + t.getMessage());
    }

    public void onComplete() {
        System.out.println("Stream finished!");
    }
};
```

Notice how the subscriber controls its own pace by calling `request(1)` — it only
asks for one item at a time. This is **backpressure** in action: the subscriber, not
the publisher, decides how fast data flows.

## Why It Matters

Every time you call `.subscribe(...)` on a `Mono` or `Flux` in Project Reactor, you
are creating a `Subscriber` under the hood. Understanding its four callbacks
(`onSubscribe`, `onNext`, `onError`, `onComplete`) explains exactly what can happen
during any reactive stream's lifecycle.
