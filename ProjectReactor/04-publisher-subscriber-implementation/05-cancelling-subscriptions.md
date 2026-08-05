# Cancelling Subscriptions

## In Simple Terms

Cancelling a subscription tells the publisher to **stop producing/sending data** and
release any resources tied to that particular subscriber. This is important for
things like infinite streams, open database cursors, or file handles — without
cancellation, they could leak resources forever.

## Simple Example

```java
Disposable subscription = Flux.interval(Duration.ofMillis(500))
    .subscribe(tick -> System.out.println("Tick #" + tick));

// Let it run for 3 seconds, then cancel
Thread.sleep(3000);
subscription.dispose(); // triggers Subscription.cancel() under the hood

System.out.println("Cancelled! No more ticks will print.");
```

You can also react to cancellation explicitly with `.doOnCancel()`:

```java
Flux.interval(Duration.ofSeconds(1))
    .doOnCancel(() -> System.out.println("Cleanup: releasing resources"))
    .subscribe(tick -> System.out.println("Tick: " + tick));
```

## Why It Matters

In a web application, if a client disconnects mid-request (closes the browser tab,
network drops), Spring WebFlux automatically cancels the underlying subscription so
the server doesn't waste CPU/memory continuing to process a response nobody will
receive. This automatic cancellation propagation is one of reactive programming's
underrated efficiency wins.
