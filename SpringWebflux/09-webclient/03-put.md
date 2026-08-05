# PUT (WebClient)

## In Simple Terms

A `PUT` request via `WebClient` updates an existing remote resource, following the
same fluent API pattern as GET/POST, just with a different HTTP method and typically
a request body containing the updated data.

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

Handling a potential `404 Not Found` from the remote service explicitly:

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

Consistent use of `WebClient`'s fluent API across all HTTP methods (GET, POST, PUT,
DELETE) keeps your inter-service communication code uniform and predictable — the
same patterns for building the request, handling errors, and extracting the response
body apply regardless of which HTTP verb you're using.
