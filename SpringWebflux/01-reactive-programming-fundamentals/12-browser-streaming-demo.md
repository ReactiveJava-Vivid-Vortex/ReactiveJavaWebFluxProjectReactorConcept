# Browser Streaming Demo

## In Simple Terms

One of the most compelling ways to *see* reactive programming in action is to stream
data directly to a browser and watch it arrive incrementally, in real time — rather
than waiting for the entire response to be ready before anything is displayed.

## Simple Example

A WebFlux endpoint streaming data every second:

```java
@GetMapping(value = "/stream-numbers", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<Long> streamNumbers() {
    return Flux.interval(Duration.ofSeconds(1))
        .map(i -> i + 1);
}
```

If you open `http://localhost:8080/stream-numbers` directly in a browser, you'll see
numbers appear one at a time, once per second, rather than the browser waiting
indefinitely and showing nothing until a final response arrives.

Comparing with a traditional (non-streaming) blocking endpoint:

```java
@GetMapping("/all-numbers-at-once")
public List<Long> allNumbersAtOnce() {
    // Blocks until ALL numbers are ready, then sends everything as one response
    return LongStream.rangeClosed(1, 10).boxed().collect(Collectors.toList());
}
```

## Why It Matters

This simple demo makes the abstract idea of "streaming" concrete and visible —
useful for explaining reactive concepts to teammates, and foundational for real
features like live dashboards, progress indicators, and Server-Sent Events (SSE)
covered later in this course.
