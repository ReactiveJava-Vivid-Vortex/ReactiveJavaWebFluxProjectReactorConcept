# Timeouts (WebClient)

## In Simple Terms

Just like any other `Mono`/`Flux`, a `WebClient` call can be bounded with
`.timeout(Duration)` to ensure it fails fast rather than waiting indefinitely for a
slow or unresponsive downstream service. Additionally, `WebClient`'s underlying HTTP
client (Netty by default) supports connection-level and response-level timeout
configuration.

## Simple Example

Applying a timeout to the reactive chain itself:

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

Configuring connection and response timeouts at the client-builder level (applies to
every call made with this `WebClient` instance):

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

Without timeouts, a single unresponsive downstream dependency can cause requests to
hang indefinitely, eventually exhausting connection pools and degrading the entire
application. Configuring both call-level (`.timeout()`) and client-level (connection/
response) timeouts is essential resilience practice for any production
service-to-service communication.
