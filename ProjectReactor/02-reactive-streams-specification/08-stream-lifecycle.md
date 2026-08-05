# Stream Lifecycle

## In Simple Terms

Every reactive stream follows a predictable lifecycle, defined by the Reactive
Streams specification. Understanding this sequence tells you exactly what signals
can happen, and in what order:

```
1. subscribe()     -> Subscriber asks to receive data from a Publisher
2. onSubscribe(s)  -> Publisher hands the Subscriber a Subscription
3. request(n)      -> Subscriber asks for n items
4. onNext(item)    -> Publisher sends items, 0 or more times
5. onComplete()    -> Publisher signals successful completion
      OR
   onError(t)      -> Publisher signals a failure
```

**Key rule:** after `onComplete()` or `onError()`, **no further signals** are sent.
The stream is finished — either successfully or with a failure, never both.

## Simple Example

```java
Flux.just("a", "b", "c")
    .subscribe(
        item -> System.out.println("onNext: " + item),
        error -> System.out.println("onError: " + error),
        () -> System.out.println("onComplete!")
    );

// Output:
// onNext: a
// onNext: b
// onNext: c
// onComplete!
```

If an error occurred instead, you'd see `onNext` calls (zero or more), then a single
`onError` call, and **no** `onComplete()` afterward.

## Why It Matters

Knowing this strict lifecycle helps you reason about your pipelines: you can rely on
the fact that once a stream errors or completes, nothing more will come through. This
is critical when writing cleanup logic (`doFinally()`) or tests (`StepVerifier`),
since you always know exactly how a stream is allowed to end.
