# Publisher

## In Simple Terms

A `Publisher` is anything that **produces a stream of data over time** and can send
that data to whoever subscribes to it. It's the source — like a TV channel
broadcasting a show. On its own, a `Publisher` does nothing; it only starts producing
data once someone (a `Subscriber`) subscribes to it.

In the Reactive Streams specification (`org.reactivestreams.Publisher`), it has one
single method:

```java
public interface Publisher<T> {
    void subscribe(Subscriber<? super T> s);
}
```

## Simple Example

```java
Publisher<Integer> publisher = subscriber -> {
    subscriber.onSubscribe(new Subscription() {
        public void request(long n) {
            subscriber.onNext(1);
            subscriber.onNext(2);
            subscriber.onNext(3);
            subscriber.onComplete();
        }
        public void cancel() { }
    });
};

publisher.subscribe(new Subscriber<Integer>() {
    public void onSubscribe(Subscription s) { s.request(Long.MAX_VALUE); }
    public void onNext(Integer item) { System.out.println("Got: " + item); }
    public void onError(Throwable t) { t.printStackTrace(); }
    public void onComplete() { System.out.println("Done!"); }
});
```

## Why It Matters

`Mono` and `Flux` in Project Reactor are both implementations of `Publisher`. Every
reactive pipeline you write starts with a `Publisher` producing data — understanding
this interface is understanding the root of the entire reactive model.
