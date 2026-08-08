# Browser Streaming Demo

## In Simple Terms

One of the best ways to actually *see* reactive programming in action is to
stream data straight to a browser and watch it show up bit by bit, in real
time — instead of waiting for the whole response to be ready before
anything appears at all.

## Simple Example

A WebFlux endpoint streaming data every second:

```java
@GetMapping(value = "/stream-numbers", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<Long> streamNumbers() {
    return Flux.interval(Duration.ofSeconds(1))
        .map(i -> i + 1);
}
```

If you open `http://localhost:8080/stream-numbers` right in your browser,
you'll see numbers appear one at a time, once a second, instead of the
browser sitting there blank until one final response arrives.

Compare that with a traditional, non-streaming blocking endpoint:

```java
@GetMapping("/all-numbers-at-once")
public List<Long> allNumbersAtOnce() {
    // Blocks until ALL numbers are ready, then sends everything as one response
    return LongStream.rangeClosed(1, 10).boxed().collect(Collectors.toList());
}
```

## Why It Matters

This simple demo makes the abstract idea of "streaming" easy to actually
see — great for explaining reactive concepts to teammates, and it's the
foundation for real features like live dashboards, progress bars, and
Server-Sent Events, which come up later in this course.
