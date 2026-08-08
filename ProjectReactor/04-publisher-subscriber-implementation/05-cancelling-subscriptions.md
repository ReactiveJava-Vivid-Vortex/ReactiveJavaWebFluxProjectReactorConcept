# Cancelling Subscriptions

## In Simple Terms

Cancelling tells the publisher to **stop sending data** and clean up anything it
was holding onto for that subscriber. This matters a lot for things like endless
streams, open database cursors, or open files — without cancelling, they could
just leak forever.

## Simple Example

```java
Disposable subscription = Flux.interval(Duration.ofMillis(500))
    .subscribe(tick -> System.out.println("Tick #" + tick));

// Let it run for 3 seconds, then cancel
Thread.sleep(3000);
subscription.dispose(); // triggers Subscription.cancel() under the hood

System.out.println("Cancelled! No more ticks will print.");
```

You can also hook into cancellation directly with `.doOnCancel()`:

```java
Flux.interval(Duration.ofSeconds(1))
    .doOnCancel(() -> System.out.println("Cleanup: releasing resources"))
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

## Why It Matters

If a user closes their browser tab mid-request, Spring WebFlux automatically
cancels the subscription behind that request — so the server doesn't keep
burning CPU and memory building a response nobody will ever see. This automatic
cancellation is one of the underrated ways reactive programming saves resources.
