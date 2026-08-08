# refCount()

## In Simple Terms

`.refCount()` (used on a `ConnectableFlux`, usually right after
`.publish()`) automatically starts the source the moment the first
subscriber shows up, and shuts it down the moment the last one leaves —
like a motion-sensor light that turns on when someone walks in and off when
the room is empty. In fact, `.share()` is really just shorthand for
`.publish().refCount()`.

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

There's also `.refCount(minSubscribers)`, which waits for a minimum number
of subscribers before it even starts — useful for coordinating several
consumers before firing up an expensive shared resource.

## Why It Matters

`.refCount()` takes the manual work out of connection management — no more
calling `.connect()`/`.dispose()` yourself. The source starts naturally when
it's needed and cleans itself up once nobody's listening, which is exactly
the convenience `.share()` bundles up for you.
