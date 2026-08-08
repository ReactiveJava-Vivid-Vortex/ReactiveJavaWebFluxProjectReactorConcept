# windowTimeout()

## In Simple Terms

`.windowTimeout()` is to `.window()` what `.bufferTimeout()` is to
`.buffer()` — it closes off a window either once it fills up, or once a
time limit runs out, whichever comes first. Each window is still a live
`Flux`, not a pre-packed list.

## Simple Example

```java
Flux.interval(Duration.ofMillis(100))
    .windowTimeout(5, Duration.ofMillis(300))
    .flatMap(Flux::collectList)
    .subscribe(batch -> System.out.println("Window: " + batch));
```

Output (windows close roughly every 300ms, so they contain around 3 items
instead of waiting to fill up to 5):
```
Window: [0, 1, 2]
Window: [3, 4, 5]
Window: [6, 7, 8]
...
```

## Why It Matters

`.windowTimeout()` gives you the flexibility of streaming windows plus a
time-based safety net — great for grouping a continuous, irregular stream
(like sensor readings or log events) into manageable, reactive chunks
without ever waiting endlessly for a size-based window to fill.
