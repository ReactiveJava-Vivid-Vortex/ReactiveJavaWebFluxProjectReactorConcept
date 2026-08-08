# Subscription Model

## In Simple Terms

Here's how data actually starts flowing: everything begins at the very end, with
`.subscribe()`, and works its way **backward first**. The request for data
travels **upstream**, from subscriber all the way back to the source. Only then
does the actual data travel **downstream**, from source back to subscriber.

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

Subscribing and asking for data happen first (traveling all the way up to the
source), and only then do the actual items travel back down, one at a time.

## Why It Matters

Once you see that data flows "up then down," it explains why nothing in a
pipeline runs until `.subscribe()` is called at the very end — and why requests
for more data always start at the consumer and move toward the source, never the
other way around.
