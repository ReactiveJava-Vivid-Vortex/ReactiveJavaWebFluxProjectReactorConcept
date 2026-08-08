# FluxSink

## In Simple Terms

`FluxSink<T>` is the object you use inside `Flux.create()` or `Flux.push()` to
manually push items into a stream. Unlike `MonoSink` (which only lets you emit
once), `FluxSink` lets you call `sink.next(value)` **as many times as you want**,
then finish up with `sink.complete()` or `sink.error(throwable)`.

```java
public interface FluxSink<T> {
    FluxSink<T> next(T t);
    void complete();
    void error(Throwable e);
    // + overflow strategy, cancellation hooks, requestedFromDownstream(), etc.
}
```

## Simple Example

```java
Flux<Integer> flux = Flux.create(sink -> {
    for (int i = 1; i <= 5; i++) {
        sink.next(i);
    }
    sink.complete();
});

flux.subscribe(value -> System.out.println("Got: " + value));
```

You can also check how much demand is outstanding, to avoid producing too much:

```java
Flux.create(sink -> {
    while (sink.requestedFromDownstream() > 0 && hasMoreData()) {
        sink.next(getNextItem());
    }
});
```

## Why It Matters

`FluxSink` is your manual dial for feeding external, push-based data — queue
messages, sensor readings, WebSocket frames — into a reactive pipeline. Handling
overflow correctly (`BUFFER`, `DROP`, `LATEST`, or `ERROR`) matters a lot here —
otherwise a fast producer can overwhelm a slow consumer and cause memory
trouble.
