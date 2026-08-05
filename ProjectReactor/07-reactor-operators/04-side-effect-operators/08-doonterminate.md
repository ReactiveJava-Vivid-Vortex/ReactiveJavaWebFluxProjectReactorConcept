# doOnTerminate()

## In Simple Terms

`.doOnTerminate(runnable)` runs a side effect when the stream ends either
successfully (`onComplete`) or with an error (`onError`) — but, unlike
`.doFinally()`, it does **not** run on cancellation. Think of it as "finally, but not
for cancellation."

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

Use `.doOnTerminate()` when you specifically want logic to run only on a "natural"
end (success or failure), but explicitly **not** when a subscriber simply walks away
(cancels). If you need truly universal cleanup — including on cancellation — prefer
`.doFinally()` instead, since forgetting that distinction is a common source of
resource leaks in cancellable streams (like an HTTP request that a client aborts).
