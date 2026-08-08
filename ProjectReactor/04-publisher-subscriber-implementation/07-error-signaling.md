# Error Signaling

## In Simple Terms

Error signaling is simply how a publisher says "something went wrong" — by
calling `onError(throwable)`. It's the reactive version of throwing an exception,
except the error travels down the pipeline like a normal signal, and any operator
along the way can catch and react to it.

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

Notice `4` never gets processed — once `onError` fires, the stream is over.
There's no going back to finish the rest.

## Why It Matters

Because an error is just another signal flowing through the pipeline, you can
catch and react to it mid-stream with operators like `onErrorResume()` or
`onErrorReturn()` — something a regular try/catch around async code can't easily
do. This is the biggest reason reactive error handling feels different from
traditional exception handling, and once you get used to it, more useful too.
