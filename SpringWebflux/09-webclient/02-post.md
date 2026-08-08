# POST (WebClient)

## In Simple Terms

A POST request through `WebClient` sends data to another service to
create something new — you supply the request body (a plain object, or a
`Mono`/`Flux` for a reactive body), and get the response back as a `Mono`.

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

Using a reactive `Mono` request body instead (handy when the body itself
comes from an upstream async source):

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

`WebClient`'s POST support fits naturally into the rest of a reactive
pipeline — you can pass an upstream `Mono` straight in as the request
body, keeping the whole chain (from handling your own request, to calling
a downstream service) non-blocking end-to-end.
