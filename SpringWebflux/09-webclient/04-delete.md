# DELETE (WebClient)

## In Simple Terms

A DELETE request through `WebClient` removes a resource on another
service. Since there's usually nothing meaningful to send back, the result
is typically a `Mono<Void>`.

## Simple Example

```java
public Mono<Void> deleteUser(String id) {
    return webClient.delete()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(Void.class);
}
```

Checking the response status explicitly, without needing a body:

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

Correctly treating a DELETE call's result as `Mono<Void>` (instead of
trying to parse a body that doesn't exist) keeps your service method
signatures accurate and avoids runtime errors from trying to deserialize
an empty response.
