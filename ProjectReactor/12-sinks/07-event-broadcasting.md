# Event Broadcasting

## In Simple Terms

Event broadcasting is the overall pattern of using a `Sinks.Many` (usually
multicast) to let multiple independent parts of your app know when
something happened — a lightweight, in-process version of publish/subscribe,
built entirely out of Reactor pieces.

## Simple Example

```java
public class OrderEventBus {
    private final Sinks.Many<OrderEvent> sink = Sinks.many().multicast().onBackpressureBuffer();

    public void publish(OrderEvent event) {
        Sinks.EmitResult result = sink.tryEmitNext(event);
        if (result.isFailure()) {
            log.warn("Failed to emit event: {}", result);
        }
    }

    public Flux<OrderEvent> subscribeToEvents() {
        return sink.asFlux();
    }
}

// Usage - multiple independent subscribers reacting to the same events:
OrderEventBus bus = new OrderEventBus();

bus.subscribeToEvents().subscribe(event -> emailService.notify(event));
bus.subscribeToEvents().subscribe(event -> auditLogger.log(event));
bus.subscribeToEvents().subscribe(event -> analyticsService.track(event));

bus.publish(new OrderEvent("ORD-123", "CREATED"));
// All three subscribers react independently to the same event
```

## Why It Matters

Event broadcasting through `Sinks` lets you keep the thing that *produces*
an event (say, an order service) separate from the many things that
*react* to it (email, audit, analytics) — each one can just subscribe and
do its own thing, without the producer ever needing to know they exist.
It's a foundational trick for building reactive systems where the pieces
don't need to know much about each other.
