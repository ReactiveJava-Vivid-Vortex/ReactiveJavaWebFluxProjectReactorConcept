# doOnRequest()

## In Simple Terms

`.doOnRequest()` lets you watch every time the downstream side asks for more
items ("send me `n` more") at this point in the pipeline. It's a diagnostic
window into exactly how much a consumer is asking for, and when.

## Simple Example

```java
Flux.range(1, 5)
    .doOnRequest(n -> System.out.println("Downstream requested: " + n))
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(2);
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println("Got: " + value);
            request(1);
        }
    });
```

Output:
```
Downstream requested: 2
Got: 1
Downstream requested: 1
Got: 2
Downstream requested: 1
Got: 3
...
```

## Why It Matters

`.doOnRequest()` is a lifesaver when you're debugging a backpressure
mystery — like figuring out why a slow consumer seems stuck, or checking
that a custom subscriber is really asking for data in the batch sizes you
expect it to.
