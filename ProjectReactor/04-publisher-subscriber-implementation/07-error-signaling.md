# Error Signaling

## In Simple Terms

Error signaling is how a publisher communicates that something went wrong, via a
single `onError(Throwable)` call. This is the reactive equivalent of throwing an
exception — except instead of unwinding the call stack synchronously, the error
travels downstream as a normal signal through the pipeline, and can be intercepted
and handled by operators along the way.

## Simple Example

```java
Flux<Integer> risky = Flux.just(1, 2, 0, 4)
    .map(n -> {
        if (n == 0) throw new ArithmeticException("Cannot process zero!");
        return 10 / n;
    });

risky.subscribe(
    value -> System.out.println("Value: " + value),
    error -> System.out.println("Caught error: " + error.getMessage())
);
```

Output:
```
Value: 10
Value: 5
Caught error: Cannot process zero!
```

Notice `4` is never processed — once `onError` fires, the stream is terminated; there
is no going back to processing more items.

## Why It Matters

Because errors are just another kind of signal in the pipeline, you can intercept and
react to them mid-stream using operators like `onErrorResume()` or `onErrorReturn()`
— something a plain `try/catch` around asynchronous code cannot easily do. This is a
core reason reactive error handling feels different (and, once learned, more
powerful) than traditional exception handling.
