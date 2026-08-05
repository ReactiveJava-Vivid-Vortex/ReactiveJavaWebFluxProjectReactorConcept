# onError()

## In Simple Terms

`onError(Throwable t)` is the signal a `Publisher` sends when **something went
wrong** and the stream must terminate. Like `onComplete()`, it's a *terminal* signal
— once called, no more `onNext()` or `onComplete()` will follow.

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
// (note: "4" is never processed — the stream terminated at the error)
```

## Why It Matters

Unlike traditional try/catch, if you **don't** provide an `onError` handler in a
reactive pipeline, the error doesn't just vanish — Reactor will throw an
`UnsupportedOperatorException` warning you forgot to handle it (or, in some contexts,
propagate it as an unhandled exception). This is why operators like `onErrorResume()`,
`onErrorReturn()`, and providing an explicit error consumer in `subscribe()` are so
important — they are your only chance to react to failures in a reactive pipeline.
