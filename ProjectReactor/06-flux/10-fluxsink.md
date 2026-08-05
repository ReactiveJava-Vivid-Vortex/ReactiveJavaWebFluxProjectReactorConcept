# FluxSink

## In Simple Terms

`FluxSink<T>` is the object you use inside `Flux.create()` or `Flux.push()` to
manually emit items into the stream. Unlike `MonoSink` (which allows only one
emission), `FluxSink` lets you call `sink.next(value)` **many times**, followed by
either `sink.complete()` or `sink.error(throwable)` to end the stream.

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

You can also check downstream demand to avoid overproducing:

```java
Flux.create(sink -> {
    while (sink.requestedFromDownstream() > 0 && hasMoreData()) {
        sink.next(getNextItem());
    }
});
```

## Why It Matters

`FluxSink` is your manual control point for feeding external, push-based data (e.g.,
messages from a queue, sensor readings, or WebSocket frames) into a reactive
pipeline. Handling overflow correctly (via `OverflowStrategy.BUFFER`, `DROP`,
`LATEST`, or `ERROR`) is critical — otherwise a fast producer can overwhelm a slow
consumer and cause memory issues.
