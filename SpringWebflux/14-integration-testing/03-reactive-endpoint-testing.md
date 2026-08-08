# Reactive Endpoint Testing

## In Simple Terms

Testing reactive endpoints has a few extra wrinkles compared to
traditional Spring MVC testing — mostly around testing streaming responses
(a `Flux` with NDJSON/SSE media types), which `WebTestClient` supports
through its `.returnResult()` and `FluxExchangeResult` APIs.

## Simple Example

Testing a streaming (NDJSON) endpoint:

```java
@Test
void streamProducts_returnsMultipleItems() {
    FluxExchangeResult<ProductDto> result = webTestClient.get()
        .uri("/products/stream")
        .accept(MediaType.APPLICATION_NDJSON)
        .exchange()
        .expectStatus().isOk()
        .returnResult(ProductDto.class);

    StepVerifier.create(result.getResponseBody())
        .expectNextCount(3)
        .verifyComplete();
}
```

Testing an SSE endpoint the same way:

```java
@Test
void streamNotifications_emitsEvents() {
    FluxExchangeResult<String> result = webTestClient.get()
        .uri("/notifications")
        .accept(MediaType.TEXT_EVENT_STREAM)
        .exchange()
        .expectStatus().isOk()
        .returnResult(String.class);

    StepVerifier.create(result.getResponseBody())
        .expectNextCount(2)
        .thenCancel() // don't wait for an infinite stream to "complete"
        .verify();
}
```

## Why It Matters

Pairing `WebTestClient` (for the HTTP side) with `StepVerifier` (for
checking the streamed body) gives you complete coverage over both the
request/response contract and the actual streaming behavior — essential
for properly testing endpoints that stream large or ongoing data (see
[[reactive-streaming]] and [[server-sent-events-sse]]).
