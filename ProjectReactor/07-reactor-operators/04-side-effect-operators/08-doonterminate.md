# doOnTerminate()

## In Simple Terms

`.doOnTerminate()` runs when the stream ends naturally — either it finished
successfully or it failed with an error — but, unlike `.doFinally()`, it
does **not** run if someone just walks away and cancels the subscription.
Think of it as "finally, minus the cancellation case."

## Simple Example

```java
Flux.just(1, 2, 3)
    .doOnTerminate(() -> System.out.println("Terminated (success or error)"))
    .subscribe(n -> System.out.println("Item: " + n));
```

Output:
```
Item: 1
Item: 2
Item: 3
Terminated (success or error)
```

## doOnTerminate() vs doFinally()

| Scenario     | doOnTerminate() | doFinally() |
|--------------|:---------------:|:-----------:|
| onComplete   | ✅ runs          | ✅ runs      |
| onError      | ✅ runs          | ✅ runs      |
| cancellation | ❌ does NOT run  | ✅ runs      |

## Why It Matters

Reach for `.doOnTerminate()` when you want logic to run only on a "natural"
ending (success or failure), but you deliberately do **not** want it running
just because a subscriber cancelled. If you need cleanup that truly always
happens — cancellation included — use `.doFinally()` instead. Mixing these
two up is a common way resource leaks sneak into cancellable streams, like
an HTTP request the client gives up on.
