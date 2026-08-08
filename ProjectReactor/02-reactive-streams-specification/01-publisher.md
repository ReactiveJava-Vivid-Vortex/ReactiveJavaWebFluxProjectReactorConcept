# Publisher

## In Simple Terms

A `Publisher` is anything that hands out data over time to whoever asks for it —
like a TV channel broadcasting a show. It doesn't do anything on its own. It only
starts sending data once someone (a `Subscriber`) tunes in.

The whole interface is just one method:

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

`Mono` and `Flux` are both just `Publisher`s under the hood. Every reactive
pipeline you write starts with a `Publisher` producing data — so this one
interface is really the root of everything else in this course.
