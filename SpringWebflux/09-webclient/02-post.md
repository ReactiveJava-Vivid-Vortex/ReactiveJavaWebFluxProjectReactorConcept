# POST (WebClient)

## In Simple Terms

A `POST` request via `WebClient` sends data to a remote service to create a new
resource — you supply the request body (either a plain object or a `Mono`/`Flux`
for a reactive body), and get the response back as a `Mono`.

## Simple Example

```java
public Mono<UserDto> createUser(CreateUserRequest request) {
    return webClient.post()
        .uri("/users")
        .contentType(MediaType.APPLICATION_JSON)
        .bodyValue(request) // for a plain, already-known object
        .retrieve()
        .bodyToMono(UserDto.class);
}
```

Using a reactive `Mono` request body instead (useful when the body itself comes from
an upstream async source):

```java
public Mono<UserDto> createUserFromUpstream(Mono<CreateUserRequest> requestMono) {
    return webClient.post()
        .uri("/users")
        .body(requestMono, CreateUserRequest.class)
        .retrieve()
        .bodyToMono(UserDto.class);
}
```

## Why It Matters

`WebClient`'s POST support integrates naturally with the rest of a reactive
pipeline — you can pass an upstream `Mono` directly as the request body, letting the
whole chain (from receiving your own request, to calling the downstream service)
remain non-blocking end-to-end.
