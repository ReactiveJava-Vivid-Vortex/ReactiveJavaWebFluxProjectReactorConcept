# WebClient — Topic Overview

## What Is This Topic About? (In Simple Terms)

`WebClient` is Spring's fully non-blocking HTTP client — the reactive replacement
for the old, blocking `RestTemplate`. Whenever your WebFlux service needs to call
another service over HTTP, `WebClient` is the tool, and it returns `Mono`/`Flux`
just like everything else in this ecosystem, so it composes naturally into your
pipelines.

```java
WebClient webClient = WebClient.builder().baseUrl("https://api.example.com").build();

public Mono<UserDto> getUser(String id) {
    return webClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(UserDto.class);
}
```

The same fluent pattern — `.get()/.post()/.put()/.delete()` → build the request
(`.uri()`, `.bodyValue()`) → `.retrieve()` → extract the response
(`.bodyToMono()`/`.bodyToFlux()`) — covers every HTTP verb. Choosing
`bodyToMono()` vs `bodyToFlux()` matters: use `bodyToMono()` for 0-1 results, or
you'll silently only capture the first item of a multi-item response.

By default, any 4xx/5xx status becomes a `WebClientResponseException` — use
`.onStatus()` to translate specific statuses into your own domain exceptions.
Because a blocking HTTP call would undermine everything WebFlux does, **always**
apply a `.timeout()` to every outgoing call, so one slow downstream dependency can
never hang your own service indefinitely. `WebClient` filters
(`ExchangeFilterFunction`) let you centralize cross-cutting concerns (auth headers,
logging) across every call made through a shared client instance.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **GET** | `.get().uri(...).retrieve().bodyToMono/Flux(Class)` — retrieve data reactively, no blocking. |
| 2 | **POST** | `.post().bodyValue(obj)` (or `.body(mono, Class)` for a reactive body) to create a remote resource. |
| 3 | **PUT** | Same fluent pattern as POST, different verb — updates an existing remote resource. |
| 4 | **DELETE** | `.delete().uri(...).retrieve().bodyToMono(Void.class)` — no meaningful response body expected. |
| 5 | **Request Body** | `.bodyValue(obj)` for values you have; `.body(mono/flux, Class)` for reactive/streaming bodies. |
| 6 | **Response Body** | `.bodyToMono()` for 0-1 results, `.bodyToFlux()` for 0-N — picking wrong silently drops data. |
| 7 | **Error Handling** | Default: 4xx/5xx → `WebClientResponseException`; use `.onStatus()` to map to custom domain exceptions. |
| 8 | **Timeouts** | Always `.timeout(Duration)` outgoing calls — an unresponsive dependency should never hang your service forever. |
| 9 | **Filters** | `ExchangeFilterFunction` centralizes cross-cutting concerns (auth headers, logging) across all calls on a client. |
| 10 | **Exchange strategies** | Configure encoding/decoding, notably the default 256KB in-memory response buffer limit (`maxInMemorySize`). |

## How It All Fits Together

```
webClient.method()               ← GET / POST / PUT / DELETE
    .uri(...)
    .bodyValue(...) / .body(mono, Class)   ← Request Body (if applicable)
    .retrieve()
    .onStatus(...)                ← Error Handling (map 4xx/5xx to domain exceptions)
    .bodyToMono/Flux(Class)        ← Response Body
    .timeout(Duration.ofSeconds(n)) ← ALWAYS bound the wait
```

Treat every `WebClient` call like a potential failure point: pick the right body
extraction method, map errors deliberately, and never skip the timeout — this is
the exact pattern used throughout the Real-World Microservice Scenarios topic later
in this course.
