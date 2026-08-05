# Event Generation

## In Simple Terms

"Event generation" refers to the pattern of producing a continuous or periodic
sequence of application-level events (e.g., "new order created," "temperature
updated," "price changed") as a `Flux`, so that any interested subscriber can react
to them as they happen, in real time.

## Simple Example

A simple in-memory event bus using a `Flux`:

```java
public class OrderEventPublisher {
    private final Sinks.Many<String> sink = Sinks.many().multicast().onBackpressureBuffer();

    public void publishOrderCreated(String orderId) {
        sink.tryEmitNext("Order created: " + orderId);
    }

    public Flux<String> events() {
        return sink.asFlux();
    }
}

// Usage:
OrderEventPublisher publisher = new OrderEventPublisher();
publisher.events().subscribe(event -> System.out.println("Listener received: " + event));

publisher.publishOrderCreated("ORD-123");
// Listener received: Order created: ORD-123
```

(Note: `Sinks` — covered later — are the modern, recommended way to generate events
manually rather than raw `Processor` implementations.)

## Why It Matters

Event generation via `Flux` is the foundation for reactive features like live
dashboards, Server-Sent Events (SSE), and internal pub/sub between components — any
place where "something happened, and multiple parts of the system should react"
applies.
