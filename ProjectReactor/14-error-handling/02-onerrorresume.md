# onErrorResume()

## In Simple Terms

`.onErrorResume()` catches an error and switches over to a completely
different `Mono`/`Flux` — which can itself go do more async work, like
calling a backup service. It's more powerful than `.onErrorReturn()`
because your "plan B" can be a full operation, not just a static value.

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

Matching specific error types for targeted recovery:

```java
callExternalService()
    .onErrorResume(TimeoutException.class, e -> Mono.just("Fallback: service timed out"))
    .onErrorResume(IOException.class, e -> Mono.just("Fallback: network issue"))
    .subscribe(System.out::println);
```

## Why It Matters

`.onErrorResume()` is the go-to tool for resilience in microservices —
falling back to a cache, a secondary service, or a scaled-down but working
response when the main dependency fails, all written as a natural part of
the chain instead of a separate try/catch block.
