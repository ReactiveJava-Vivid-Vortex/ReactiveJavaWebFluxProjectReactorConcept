# Q1. What Is WebClient, and Why Not Just Use RestTemplate?

## Simple Explanation (Think of a Phone Call vs Leaving a Voicemail and Waiting by the Phone)

`RestTemplate` (blocking) is like calling someone and **standing there in
silence** until they answer — you can't do anything else meanwhile. `WebClient`
(non-blocking) is like leaving a message and immediately going back to your other
work — you get notified the moment they call back.

```java
WebClient webClient = WebClient.builder().baseUrl("https://api.example.com").build();

public Mono<UserDto> getUser(String id) {
    return webClient.get()
        .uri("/users/{id}", id)
        .retrieve()
        .bodyToMono(UserDto.class); // returns IMMEDIATELY — no thread frozen waiting
}
```

Using `RestTemplate` inside a WebFlux app would freeze an event-loop thread —
exactly the mistake WebFlux exists to avoid.

---

## Q2. What Does the Fluent Pattern Look Like for Every HTTP Verb?

```java
webClient.get().uri("/users/{id}", id).retrieve().bodyToMono(UserDto.class);
webClient.post().uri("/users").bodyValue(request).retrieve().bodyToMono(UserDto.class);
webClient.put().uri("/users/{id}", id).bodyValue(request).retrieve().bodyToMono(UserDto.class);
webClient.delete().uri("/users/{id}", id).retrieve().bodyToMono(Void.class);
```

Same shape every time: **method → uri → (body) → retrieve → extract response.**

---

## Q3. `bodyToMono()` vs `bodyToFlux()` — Choosing Wrong Silently Drops Data

```java
// If the API actually returns MULTIPLE users but you use bodyToMono()...
Mono<UserDto> oneUser = webClient.get().uri("/users").retrieve().bodyToMono(UserDto.class);
// ...you silently only get the FIRST one — no error, no warning!

// Correct, if the endpoint returns many:
Flux<UserDto> allUsers = webClient.get().uri("/users").retrieve().bodyToFlux(UserDto.class);
```

Always match the extraction method to the actual cardinality of the response.

---

## Q4. How Do I Handle Errors from a Downstream Call?

```java
webClient.get().uri("/users/{id}", id)
    .retrieve()
    .onStatus(HttpStatusCode::is4xxClientError, response ->
        response.bodyToMono(String.class).flatMap(body -> Mono.error(new ClientException(response.statusCode(), body)))
    )
    .onStatus(HttpStatusCode::is5xxServerError, response -> Mono.error(new DownstreamServiceException("Remote service failed")))
    .bodyToMono(UserDto.class);
```

Without `.onStatus()`, any 4xx/5xx becomes a generic `WebClientResponseException`
— translate it into a meaningful domain exception instead.

---

## Q5. Why Is `.timeout()` Non-Negotiable on Every Outgoing Call?

```java
webClient.get().uri("/users/{id}", id).retrieve().bodyToMono(UserDto.class)
    .timeout(Duration.ofSeconds(2)) // NEVER skip this
    .onErrorResume(TimeoutException.class, e -> Mono.just(UserDto.placeholder()));
```

Without a timeout, one unresponsive downstream dependency can hang your own
service indefinitely — the exact failure mode covered in depth in the Real-World
Microservice Scenarios topic.

---

## Q6. How Do I Centralize Cross-Cutting Concerns (Auth Headers, Logging)?

```java
ExchangeFilterFunction addAuthHeader = ExchangeFilterFunction.ofRequestProcessor(request ->
    Mono.just(ClientRequest.from(request).header("Authorization", "Bearer " + getToken()).build())
);

WebClient webClient = WebClient.builder()
    .baseUrl("https://api.example.com")
    .filter(addAuthHeader) // applied automatically to EVERY call through this client
    .build();
```

---

## Q7. Interview-Style Q&A

### Does calling `.retrieve().bodyToMono(...)` block the calling thread?

**No** — it returns a `Mono` immediately; the actual HTTP call happens
asynchronously, non-blocking.

### What's the default in-memory buffer limit for a response body?

256KB by default — larger responses throw `DataBufferLimitException` unless you
increase it via `ExchangeStrategies`.

### If I forget `.onStatus()`, what happens on a 404?

It becomes a generic `WebClientResponseException.NotFound` — still catchable, but
less domain-meaningful than a custom mapped exception.

---

## Q8. Summary

```
webClient.method()               ← GET / POST / PUT / DELETE
    .uri(...)
    .bodyValue(...) / .body(mono, Class)   ← Request Body (if applicable)
    .retrieve()
    .onStatus(...)                ← map 4xx/5xx to domain exceptions
    .bodyToMono/Flux(Class)        ← match cardinality correctly!
    .timeout(Duration.ofSeconds(n)) ← ALWAYS bound the wait
```

### One sentence to remember

> **"WebClient is the non-blocking replacement for RestTemplate — every call
> needs a timeout, deliberate error mapping, and the right bodyToMono/Flux
> choice, or you'll silently reintroduce the exact problems WebFlux exists to
> solve."**
