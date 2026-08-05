# Reactive Endpoint Testing

## In Simple Terms

Testing reactive endpoints has a few nuances beyond traditional Spring MVC testing —
particularly around testing **streaming** responses (`Flux` with NDJSON/SSE media
types), which `WebTestClient` supports through its `.returnResult()` and
`FluxExchangeResult` APIs.

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

Testing an SSE endpoint similarly:

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

Combining `WebTestClient` (for the HTTP layer) with `StepVerifier` (for asserting on
the streamed response body) gives you full test coverage over both the request/
response contract and the actual streaming behavior — essential for correctly
verifying endpoints that stream large or continuous data (see [[reactive-streaming]]
and [[server-sent-events-sse]]).
