# refCount()

## In Simple Terms

`.refCount()` (called on a `ConnectableFlux`, typically after `.publish()`)
automatically manages the connection lifecycle based on the number of active
subscribers: it connects (starts the source) when the **first** subscriber arrives,
and disconnects (stops the source) when the **last** subscriber leaves. In fact,
`.share()` is essentially shorthand for `.publish().refCount()`.

## Simple Example

```java
Flux<Long> autoManaged = Flux.interval(Duration.ofSeconds(1))
    .doOnSubscribe(s -> System.out.println("Source started"))
    .doOnCancel(() -> System.out.println("Source stopped"))
    .publish()
    .refCount(); // equivalent to using .share() directly

Disposable sub1 = autoManaged.subscribe(t -> System.out.println("A: " + t));
Disposable sub2 = autoManaged.subscribe(t -> System.out.println("B: " + t));

// Source started only once, shared by both

sub1.dispose();
sub2.dispose(); // last subscriber leaves -> "Source stopped" prints
```

There's also `.refCount(minSubscribers)`, which waits for a minimum number of
subscribers before connecting — useful for coordinating multiple consumers before
starting an expensive shared resource.

## Why It Matters

`.refCount()` automates connection management so you don't have to manually call
`.connect()`/`.dispose()` yourself — the source naturally starts when needed and
cleans itself up when no one is listening anymore, which is exactly the behavior
`.share()` provides as a convenient shorthand.
