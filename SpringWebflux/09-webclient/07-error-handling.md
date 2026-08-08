# Error Handling (WebClient)

## In Simple Terms

By default, `WebClient`'s `.retrieve()` treats any 4xx/5xx status as a
failure, turning it into a `WebClientResponseException` sent through the
reactive error channel. You can customize this with `.onStatus()` to map
specific status codes to your own exceptions instead.

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

Handling `WebClient` errors properly — turning remote HTTP failures into
meaningful, specific exceptions — matters a lot for keeping services
resilient. Without it, a downstream 500 error might travel all the way to
your own API's clients as an opaque `WebClientResponseException`, with
none of the context they'd need to understand or recover from it.
