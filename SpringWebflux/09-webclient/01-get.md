# GET (WebClient)

## In Simple Terms

`WebClient` is Spring's fully reactive, non-blocking HTTP client — the modern
replacement for the older, blocking `RestTemplate`. A `GET` request retrieves data
from a remote service and returns it as a `Mono`/`Flux`, without ever blocking the
calling thread.

## Simple Example

```java
WebClient webClient = WebClient.builder()
    .baseUrl("https://api.example.com")
    .build();

public Mono<UserDto> getUser(String id) {
    return webClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(UserDto.class);
}

public Flux<UserDto> getAllUsers() {
    return webClient.get()
        .uri("/users")
        .retrieve()
        .bodyToFlux(UserDto.class);
}
```

## Why It Matters

Using `WebClient` (instead of blocking `RestTemplate`) inside a WebFlux application
is essential to preserving the entire non-blocking chain — a blocking HTTP call
buried inside an otherwise reactive pipeline would stall an event-loop thread (see
[[non-blocking-execution]] in the ProjectReactor notes), undermining the whole point
of using WebFlux in the first place.
