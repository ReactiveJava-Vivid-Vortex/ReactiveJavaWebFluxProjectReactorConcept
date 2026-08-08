# Stream Lifecycle

## In Simple Terms

Every reactive stream follows the same predictable script from start to finish.
Once you know the script, you always know exactly what can happen and in what
order:

```
1. subscribe()     -> Subscriber asks to receive data from a Publisher
2. onSubscribe(s)  -> Publisher hands the Subscriber a Subscription
3. request(n)      -> Subscriber asks for n items
4. onNext(item)    -> Publisher sends items, 0 or more times
5. onComplete()    -> Publisher signals successful completion
      OR
   onError(t)      -> Publisher signals a failure
```

**The one rule to remember:** once `onComplete()` or `onError()` happens,
**nothing else follows.** The stream is over — either it worked, or it failed,
but never both.

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

If something had failed instead, you'd see some `onNext` calls (maybe zero, maybe
a few), then one `onError` call — and no `onComplete()` after that.

## Why It Matters

Knowing this script by heart makes reasoning about your code much easier — once a
stream errors or finishes, you know for certain nothing more is coming. That's
exactly why cleanup logic (`doFinally()`) and tests (`StepVerifier`) can rely on
it so confidently.
