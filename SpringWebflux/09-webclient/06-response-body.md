# Response Body

## In Simple Terms

After `.retrieve()`, `WebClient` gives you a few ways to pull out the
response body: `.bodyToMono(Class)` for a single object, `.bodyToFlux(Class)`
for a stream of objects, or `.toEntity(Class)` if you need the whole
`ResponseEntity` (headers and status included).

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

Choosing the right way to read the response actually matters for
correctness: calling `.bodyToMono()` on an endpoint that really returns
multiple items would silently grab only the first one. Knowing whether the
remote API sends back 0-1 or 0-N items (and matching that to
`bodyToMono`/`bodyToFlux` accordingly) saves you from a subtle,
easy-to-miss bug.
