# Custom Publishers (Flux)

## In Simple Terms

Reactor gives you a lot of built-in ways to create a `Flux`, but sometimes you
need to wrap something unusual — a proprietary device, a third-party SDK with its
own quirks. You can do this by combining `Flux.generate()` or `Flux.create()`
with your own logic (implementing the raw `Publisher` interface directly is
almost never necessary).

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

Wrapping a weird or legacy data source behind a well-behaved `Flux` means the
rest of your code can treat it just like any other reactive stream — combining
it with `.map()`, `.filter()`, error handling, and backpressure — without
needing to know its messy internal details.
