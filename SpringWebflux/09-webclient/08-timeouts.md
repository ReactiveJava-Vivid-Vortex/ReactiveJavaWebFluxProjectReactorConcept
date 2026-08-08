# Timeouts (WebClient)

## In Simple Terms

Just like any other `Mono`/`Flux`, a `WebClient` call can be given a
`.timeout(Duration)` so it fails fast instead of waiting forever on a slow
or stuck downstream service. On top of that, `WebClient`'s underlying HTTP
client (Netty by default) also lets you configure timeouts at the
connection and response level.

## Simple Example

Applying a timeout right on the reactive chain:

```java
public Mono<UserDto> getUser(String id) {
    return webClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(UserDto.class)
        .timeout(Duration.ofSeconds(3))
        .onErrorResume(TimeoutException.class, e ->
            Mono.error(new ServiceUnavailableException("User service timed out"))
        );
}
```

Configuring connection and response timeouts at the client-builder level
(applies to every call made through this `WebClient` instance):

```java
HttpClient httpClient = HttpClient.create()
    .option(ChannelOption.CONNECT_TIMEOUT_MILLIS, 2000)
    .responseTimeout(Duration.ofSeconds(5));

WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(httpClient))
    .baseUrl("https://api.example.com")
    .build();
```

## Why It Matters

Without timeouts, one unresponsive downstream dependency can leave
requests hanging indefinitely, eventually exhausting connection pools and
dragging down the whole app. Setting both call-level (`.timeout()`) and
client-level timeouts is basic resilience hygiene for talking to other
services in production.
