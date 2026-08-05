# Custom Publishers (Flux)

## In Simple Terms

While Project Reactor gives you many built-in `Flux` factory methods, sometimes you
need a fully custom publisher tailored to a unique data source — e.g., reading from a
proprietary hardware device, or wrapping a third-party SDK with unusual semantics.
You can build this either by implementing the raw `Publisher` interface (rarely
needed) or, much more commonly, by combining `Flux.generate()` / `Flux.create()` with
your own logic.

## Simple Example

```java
public class SensorReadingPublisher {

    public static Flux<Double> readTemperatureStream(SensorDevice device) {
        return Flux.create(sink -> {
            device.onReading(reading -> sink.next(reading));
            device.onDisconnect(() -> sink.complete());
            device.onError(error -> sink.error(error));

            sink.onDispose(device::disconnect); // ensure cleanup on cancellation
        });
    }
}

// Usage:
SensorReadingPublisher.readTemperatureStream(myDevice)
    .subscribe(temp -> System.out.println("Temperature: " + temp));
```

## Why It Matters

Wrapping a proprietary or legacy data source behind a well-behaved custom `Flux`
means the rest of your codebase can treat it exactly like any other reactive stream
— composing it with `.map()`, `.filter()`, error handling, and backpressure, without
needing to know the messy details of the original source.
