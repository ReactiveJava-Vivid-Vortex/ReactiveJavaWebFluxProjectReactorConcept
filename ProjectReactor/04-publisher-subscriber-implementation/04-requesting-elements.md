# Requesting Elements

## In Simple Terms

"Requesting elements" just means the subscriber calling
`subscription.request(n)` to say "I'm ready for `n` more." This is how a
subscriber controls its own pace, and it can happen many times over the life of a
subscription — not just once at the start.

## Simple Example

```java
Flux.range(1, 10)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            System.out.println("Requesting first batch of 3");
            request(3);
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println("Processing: " + value);
            if (value % 3 == 0) {
                System.out.println("Requesting next batch of 3");
                request(3);
            }
        }
    });
```

Notice this pulls data in controlled batches of 3, instead of getting all 10
items pushed at once:

```
Requesting first batch of 3
Processing: 1
Processing: 2
Processing: 3
Requesting next batch of 3
Processing: 4
Processing: 5
Processing: 6
Requesting next batch of 3
...
```

## Why It Matters

Requesting data in small, controlled batches — instead of just asking for
`Long.MAX_VALUE` up front — is how you build a consumer that paces itself. This
matters when whatever comes next (writing to a database, calling a rate-limited
API) is slower than the source can produce data.
