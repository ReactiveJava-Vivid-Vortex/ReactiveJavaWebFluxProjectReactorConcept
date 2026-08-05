# doOnRequest()

## In Simple Terms

`.doOnRequest(consumer)` lets you observe every time downstream requests more items
(i.e., calls `request(n)`) at this point in the pipeline. It's a diagnostic tool for
understanding exactly how much demand is flowing upstream, and when.

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

`.doOnRequest()` is invaluable when debugging backpressure-related issues — e.g.,
figuring out why a slow consumer seems to be stalling, or verifying that a custom
subscriber is correctly requesting data in the batch sizes you expect.
