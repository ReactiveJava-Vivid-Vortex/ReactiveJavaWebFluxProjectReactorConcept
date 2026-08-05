# windowTimeout()

## In Simple Terms

`.windowTimeout(maxSize, maxTime)` is to `.window()` what `.bufferTimeout()` is to
`.buffer()` — it creates a new window either when `maxSize` items accumulate, or when
`maxTime` elapses, whichever comes first. Each window is still a `Flux` sub-stream,
not a pre-collected `List`.

## Simple Example

```java
Flux.interval(Duration.ofMillis(100))
    .windowTimeout(5, Duration.ofMillis(300))
    .flatMap(Flux::collectList)
    .subscribe(batch -> System.out.println("Window: " + batch));
```

Output (windows close roughly every 300ms, so they contain around 3 items instead of
waiting to fill up to 5):
```
Window: [0, 1, 2]
Window: [3, 4, 5]
Window: [6, 7, 8]
...
```

## Why It Matters

`.windowTimeout()` combines the flexibility of streaming windows (rather than
collected lists) with time-based cutoffs — ideal for grouping a continuous,
irregularly-paced stream (like sensor readings or log events) into manageable,
reactive chunks for further processing, without waiting indefinitely for a
size-based window to fill.
