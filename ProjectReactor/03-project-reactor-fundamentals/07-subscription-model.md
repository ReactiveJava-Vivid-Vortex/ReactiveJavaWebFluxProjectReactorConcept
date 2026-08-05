# Subscription Model

## In Simple Terms

The **subscription model** describes how data actually starts flowing in a reactive
pipeline: everything is driven **bottom-up**, starting from the final `.subscribe()`
call. The subscription (and the demand `request(n)`) travels **upstream** first
(from subscriber to publisher), and only then does data flow **downstream** (from
publisher to subscriber) in response.

```
subscribe() call
     |
     v
 request(n) signal travels UPSTREAM  (subscriber -> ... -> source)
     |
     v
 onNext() data travels DOWNSTREAM    (source -> ... -> subscriber)
```

## Simple Example

```java
Flux.range(1, 3)
    .doOnSubscribe(s -> System.out.println("1. Subscribed"))
    .doOnRequest(n -> System.out.println("2. Requested: " + n))
    .doOnNext(v -> System.out.println("3. Emitting: " + v))
    .subscribe(v -> System.out.println("4. Received: " + v));
```

Output:
```
1. Subscribed
2. Requested: 9223372036854775807
3. Emitting: 1
4. Received: 1
3. Emitting: 2
4. Received: 2
3. Emitting: 3
4. Received: 3
```

Notice the subscription and request signals happen first (traveling up to the
source), and only then do the actual data items flow back down, one at a time.

## Why It Matters

Understanding this bottom-up subscription flow (versus top-down data flow) explains
why nothing in a reactive pipeline runs until `.subscribe()` is called at the very
end, and why demand (backpressure) requests always originate at the consumer and
travel toward the producer — never the other way around.
