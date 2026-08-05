# Event Broadcasting

## In Simple Terms

"Event broadcasting" is the overall pattern of using a `Sinks.Many` (typically
multicast) to notify **multiple independent parts of your application** whenever
something happens — like a lightweight, in-process publish/subscribe event bus, built
entirely with Project Reactor primitives.

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

Event broadcasting via `Sinks` lets you decouple the *producer* of an event (e.g., an
order service) from its many independent *consumers* (email, audit, analytics) —
each can subscribe and react without the producer needing to know about them
individually, a foundational pattern for building loosely-coupled reactive systems.
