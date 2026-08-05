# onErrorResume()

## In Simple Terms

`.onErrorResume(fallbackFunction)` catches an error and switches to an **entirely
different Mono/Flux** (which can itself be asynchronous — e.g., a fallback service
call), based on the error that occurred. It's more powerful than `.onErrorReturn()`
because the recovery itself can be a full reactive operation.

## Simple Example

```java
public Mono<String> getWeather(String city) {
    return primaryWeatherApi.getWeather(city)
        .onErrorResume(error -> {
            System.out.println("Primary API failed: " + error.getMessage());
            return backupWeatherApi.getWeather(city); // fallback to a DIFFERENT Mono
        });
}
```

Matching specific exception types for targeted recovery:

```java
callExternalService()
    .onErrorResume(TimeoutException.class, e -> Mono.just("Fallback: service timed out"))
    .onErrorResume(IOException.class, e -> Mono.just("Fallback: network issue"))
    .subscribe(System.out::println);
```

## Why It Matters

`.onErrorResume()` is the go-to tool for **resilience patterns** in microservices —
falling back to a cache, a secondary service, or a degraded-but-functional response
when a primary dependency fails, all expressed as a natural part of the reactive
chain rather than a separate try/catch block.
