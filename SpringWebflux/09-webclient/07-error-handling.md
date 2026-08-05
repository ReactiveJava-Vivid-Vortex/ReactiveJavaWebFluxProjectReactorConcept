# Error Handling (WebClient)

## In Simple Terms

By default, `WebClient`'s `.retrieve()` treats any 4xx/5xx HTTP status as an error,
converting it into a `WebClientResponseException` delivered via the reactive
error channel. You can customize this behavior using `.onStatus()` to map specific
status codes to your own custom exceptions.

## Simple Example

Default behavior — any 4xx/5xx becomes a `WebClientResponseException`:

```java
webClient.get()
    .uri("/users/{id}", id)
    .retrieve()
    .bodyToMono(UserDto.class)
    .onErrorResume(WebClientResponseException.NotFound.class, e ->
        Mono.error(new UserNotFoundException(id))
    );
```

Custom status handling with `.onStatus()`:

```java
webClient.get()
    .uri("/users/{id}", id)
    .retrieve()
    .onStatus(HttpStatusCode::is4xxClientError, response ->
        response.bodyToMono(String.class)
            .flatMap(body -> Mono.error(new ClientException(response.statusCode(), body)))
    )
    .onStatus(HttpStatusCode::is5xxServerError, response ->
        Mono.error(new DownstreamServiceException("Remote service failed"))
    )
    .bodyToMono(UserDto.class);
```

## Why It Matters

Properly handling `WebClient` errors — mapping remote HTTP failures into meaningful
domain exceptions — is essential for building resilient service-to-service
communication; without it, a downstream 500 error might propagate as an opaque
`WebClientResponseException` all the way to your API's own clients, with none of the
context needed to understand or recover from it.
