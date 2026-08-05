# Response Body

## In Simple Terms

After `.retrieve()`, `WebClient` gives you several ways to extract the response
body: `.bodyToMono(Class)` for a single object, `.bodyToFlux(Class)` for a stream of
objects, or `.toEntity(Class)` if you need the full `ResponseEntity` (including
headers and status).

## Simple Example

```java
// Single object response
Mono<UserDto> user = webClient.get()
    .uri("/users/1")
    .retrieve()
    .bodyToMono(UserDto.class);

// Multiple objects (e.g., the remote API returns a JSON array or streams NDJSON)
Flux<UserDto> allUsers = webClient.get()
    .uri("/users")
    .retrieve()
    .bodyToFlux(UserDto.class);

// Full ResponseEntity, including status code and headers
Mono<ResponseEntity<UserDto>> fullResponse = webClient.get()
    .uri("/users/1")
    .retrieve()
    .toEntity(UserDto.class);

fullResponse.subscribe(response -> {
    System.out.println("Status: " + response.getStatusCode());
    System.out.println("Body: " + response.getBody());
});
```

## Why It Matters

Choosing the right response extraction method matters for correctness: using
`.bodyToMono()` on an endpoint that actually returns multiple items would silently
only capture the first one — understanding whether the remote API returns 0-1 or
0-N items (and matching it to `bodyToMono`/`bodyToFlux` accordingly) avoids subtle,
easy-to-miss bugs.
