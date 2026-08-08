# Subscriber

## In Simple Terms

A `Subscriber` is whoever consumes the data a `Publisher` sends out — like a
viewer watching a TV channel. It reacts to four things: getting set up, a new item
arriving, an error, or the show ending.

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

Notice this subscriber only ever asks for one item at a time by calling
`request(1)`. That's backpressure in action — the subscriber, not the publisher,
decides the pace.

## Why It Matters

Every time you call `.subscribe(...)` on a `Mono` or `Flux`, you're creating a
`Subscriber` behind the scenes. Its four callbacks — `onSubscribe`, `onNext`,
`onError`, `onComplete` — cover every single thing that can ever happen during a
reactive stream's life.
