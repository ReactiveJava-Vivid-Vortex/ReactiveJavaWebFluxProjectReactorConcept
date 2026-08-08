# Event Generation

## In Simple Terms

"Event generation" just means producing a stream of application events — "new
order created," "temperature updated," "price changed" — as a `Flux`, so anyone
interested can react to them the moment they happen.

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

(`Sinks`, covered later in this course, are the modern, recommended way to
generate events by hand — much simpler than the old raw `Processor` approach.)

## Why It Matters

Event generation via `Flux` is the foundation for things like live dashboards,
Server-Sent Events, and simple pub/sub between parts of your app — anywhere the
pattern is "something happened, and multiple things should react to it."
