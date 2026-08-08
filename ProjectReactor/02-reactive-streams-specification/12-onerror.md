# onError()

## In Simple Terms

`onError(throwable)` is the signal a publisher sends when **something broke** and
the stream has to stop. Just like `onComplete()`, it's a one-time, final signal —
once it fires, nothing else follows.

```java
public interface Subscriber<T> {
    void onError(Throwable t); // <-- terminal failure signal
}
```

## Simple Example

```java
Flux.just(1, 2, 0, 4)
    .map(n -> 10 / n) // will throw ArithmeticException when n == 0
    .subscribe(
        result -> System.out.println("Result: " + result),
        error -> System.out.println("Error occurred: " + error.getMessage())
    );

// Output:
// Result: 10
// Result: 5
// Error occurred: / by zero
// (note: "4" is never processed — the stream stopped right at the error)
```

## Why It Matters

Unlike a normal try/catch, if you don't handle `onError` in a reactive pipeline,
the error doesn't just quietly disappear — Reactor complains loudly about it. This
is exactly why operators like `onErrorResume()`, `onErrorReturn()`, and just
remembering to pass an error handler into `.subscribe()` matter so much — they're
your only chance to catch and react to a failure.
