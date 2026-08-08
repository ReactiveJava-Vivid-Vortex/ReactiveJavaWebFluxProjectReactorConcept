# PUT (WebClient)

## In Simple Terms

A PUT request through `WebClient` updates a resource on another service,
following the same fluent style as GET/POST — just a different HTTP
method, usually with a body containing the updated data.

## Simple Example

```java
public Mono<UserDto> updateUser(String id, UpdateUserRequest request) {
    return webClient.put()
        .uri("/users/{id}", id)
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(request)
        .retrieve()
        .bodyToMono(UserDto.class);
}
```

Handling a possible `404 Not Found` from the other service explicitly:

```java
public Mono<UserDto> updateUser(String id, UpdateUserRequest request) {
    return webClient.put()
        .uri("/users/{id}", id)
        .bodyValue(request)
        .retrieve()
        .onStatus(HttpStatusCode::is4xxClientError, response ->
            Mono.error(new UserNotFoundException(id))
        )
        .bodyToMono(UserDto.class);
}
```

## Why It Matters

Using `WebClient`'s fluent style consistently across GET, POST, PUT, and
DELETE keeps your inter-service calls uniform and predictable — the same
patterns for building a request, handling errors, and pulling out the
response apply no matter which HTTP verb you're using.
