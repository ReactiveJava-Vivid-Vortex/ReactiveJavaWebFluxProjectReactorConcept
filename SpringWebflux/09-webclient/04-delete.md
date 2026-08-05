# DELETE (WebClient)

## In Simple Terms

A `DELETE` request via `WebClient` removes a resource on a remote service. Since
there's usually no meaningful response body, the result is typically a
`Mono<Void>`.

## Simple Example

```java
public Mono<Void> deleteUser(String id) {
    return webClient.delete()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(Void.class);
}
```

Handling the response status explicitly, without needing a body:

```java
public Mono<Void> deleteUser(String id) {
    return webClient.delete()
        .uri("/users/{id}", id)
        .retrieve()
        .toBodilessEntity() // discards the body, just confirms success/failure
        .then();
}
```

## Why It Matters

Correctly modeling a DELETE call's result as `Mono<Void>` (rather than trying to
parse a body that doesn't exist) keeps your service layer's method signatures
accurate and prevents runtime errors from attempting to deserialize an empty
response body.
