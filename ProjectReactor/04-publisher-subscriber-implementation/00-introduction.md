# Q1. Why Build a Publisher and Subscriber by Hand?

## Simple Explanation (Think of Taking Apart an Engine)

You've been *using* `Mono`/`Flux`. This topic asks you to *become* one — implement
the raw Reactive Streams interfaces yourself, with no Reactor conveniences. It's
like taking an engine apart to see every gear, even though you'll never build one
from scratch again in real projects.

```
Using Flux.range(1, 5)          -> one line, "just works"
Hand-writing the same thing     -> reveals ALL the careful bookkeeping
                                    Reactor is doing for you invisibly
```

Once you've done this once, you deeply appreciate what `Flux.range()`,
`request(n)`, and `cancel()` are actually managing under the hood.

---

## Q2. What Does a Hand-Written Publisher Look Like?

```java
public class RangePublisher implements Publisher<Integer> {
    private final int start, count;

    @Override
    public void subscribe(Subscriber<? super Integer> subscriber) {
        subscriber.onSubscribe(new Subscription() {
            int current = start;
            int remaining = count;
            boolean cancelled = false;

            public void request(long n) {
                for (long i = 0; i < n && remaining > 0 && !cancelled; i++) {
                    subscriber.onNext(current++);
                    remaining--;
                }
                if (remaining == 0 && !cancelled) subscriber.onComplete();
            }

            public void cancel() { cancelled = true; }
        });
    }
}
```

Notice how much manual state tracking is required: `current`, `remaining`,
`cancelled` — all to correctly implement what `Flux.range(1, 5)` gives you in one
line.

---

## Q3. What Does a Hand-Written Subscriber Look Like?

```java
Subscriber<Integer> subscriber = new Subscriber<>() {
    Subscription subscription;

    public void onSubscribe(Subscription s) {
        subscription = s;
        s.request(2); // start by asking for just 2
    }

    public void onNext(Integer item) {
        System.out.println("Received: " + item);
        subscription.request(1); // ask for one more once processed
    }

    public void onError(Throwable t) { System.out.println("Error: " + t); }
    public void onComplete() { System.out.println("Done!"); }
};
```

This subscriber controls its own pace explicitly — it never has more than ~2
outstanding requested items at a time. That's backpressure, hand-implemented.

---

## Q4. What's the Trickiest Part? (The Subscription)

The `Subscription` must correctly track demand **and** handle concurrent
`request()`/`cancel()` calls safely. Getting this wrong — sending too many items,
not stopping on `cancel()` — is a classic source of subtle reactive bugs. This is
exactly why almost nobody hand-writes this in real projects.

```java
class SimpleSubscription implements Subscription {
    volatile boolean cancelled = false;
    int index = 0;

    public void request(long n) {
        if (cancelled) return;
        for (long i = 0; i < n && index < data.length; i++) {
            subscriber.onNext(data[index++]);
        }
        if (index == data.length && !cancelled) subscriber.onComplete();
    }

    public void cancel() { cancelled = true; }
}
```

---

## Q5. What Is "Demand-Driven Publishing"?

A well-behaved publisher only ever produces an item **in direct response to
`request(n)`** — never eagerly, never more than asked.

```java
Flux<Integer> demandDriven = Flux.generate(sink -> {
    System.out.println("Generating a value..."); // only runs when there's demand
    sink.next((int) (Math.random() * 100));
});

demandDriven.take(3).subscribe(v -> System.out.println("Got: " + v));
```

Output shows `"Generating a value..."` printing **exactly 3 times** — matching the
demand from `.take(3)`, never more. This is what keeps reactive streams
memory-safe even with huge or infinite sources.

---

## Q6. Interview-Style Q&A

### Why does `Reactor` give you `BaseSubscriber<T>` instead of the raw `Subscriber` interface?

Because implementing raw `Subscriber` correctly (especially the `Subscription`
side) is genuinely hard to get right — `BaseSubscriber` gives sensible defaults
while still letting you control demand manually when needed.

### What happens if a Subscription's `request()` is called concurrently from two threads?

A correct implementation must handle this safely (usually with atomic counters) —
this is exactly the kind of subtlety that makes hand-rolling a `Subscription`
risky in production code.

### If a publisher ignores `cancel()`, what happens?

It keeps emitting even though the subscriber walked away — a resource leak. A
compliant publisher must stop promptly once `cancel()` is called.

---

## Q7. Summary

| Concept | Key Takeaway |
|---|---|
| Custom Publisher | Implements `subscribe()`, hands out a `Subscription` obeying demand |
| Custom Subscriber | Implements all 4 callbacks, controls its own request pace |
| Subscription | The trickiest part — must track demand & handle `cancel()` correctly, even concurrently |
| Demand-driven publishing | Never produce ahead of demand — this is what makes reactive memory-safe |

### One sentence to remember

> **"Everything you'd have to painstakingly hand-code here — demand tracking,
> cancellation, correct signal ordering — is exactly what Flux.range(), Flux.create(),
> and every Reactor operator already does for you, correctly, for free."**
