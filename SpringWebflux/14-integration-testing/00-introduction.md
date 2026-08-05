# Integration Testing — Topic Overview

## What Is This Topic About? (In Simple Terms)

Unit tests check one class in isolation with mocked dependencies. **Integration
tests** verify that your controller, service, repository, and (ideally) a real
database all work correctly *together*. For WebFlux, the essential tool for this is
`WebTestClient` — Spring's reactive-aware equivalent of `MockMvc` — which sends real
HTTP requests to your running application and lets you assert on the response with
clean, readable syntax:

```java
@Autowired
private WebTestClient webTestClient;

@Test
void getProduct_returnsProduct() {
    webTestClient.get()
        .uri("/products/{id}", "P123")
        .exchange()
        .expectStatus().isOk()
        .expectBody(ProductDto.class)
        .value(product -> assertThat(product.name()).isEqualTo("Widget"));
}
```

`WebTestClient` handles the reactive complexity internally — subscribing to your
controller's `Mono`/`Flux` and blocking in a test-appropriate, controlled way — so
you don't need to manually manage subscriptions in your test code.

The one nuance beyond typical Spring MVC testing is verifying **streaming**
responses (NDJSON, SSE): `WebTestClient` returns a `FluxExchangeResult`, and you
combine it with `StepVerifier` to assert on the streamed body itself, since a
streaming endpoint's response is a `Flux`, not a single object.

```java
FluxExchangeResult<ProductDto> result = webTestClient.get()
    .uri("/products/stream").accept(MediaType.APPLICATION_NDJSON)
    .exchange().expectStatus().isOk().returnResult(ProductDto.class);

StepVerifier.create(result.getResponseBody()).expectNextCount(3).verifyComplete();
```

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **WebTestClient** | Spring's dedicated WebFlux test client — send real HTTP requests, assert on status/body cleanly. |
| 2 | **Integration Tests** | Verify controller + service + repository (+ real/containerized DB) work together, not just one class in isolation. |
| 3 | **Reactive endpoint testing** | For streaming (NDJSON/SSE) endpoints, combine `WebTestClient.returnResult()` + `StepVerifier` to assert the streamed body. |

## How It All Fits Together

```
Test sends request  ──▶  webTestClient.get()/.post()/...
        │
        ▼
.exchange()   (fires the request through the real application stack)
        │
        ▼
┌───────────────────────┬──────────────────────────────┐
│  Normal JSON response  │   Streaming (NDJSON/SSE) resp │
│  .expectBody(Class)    │   .returnResult(Class)         │
│  .value(assertions)    │   → StepVerifier.create(body)  │
└───────────────────────┴──────────────────────────────┘
```

Treat `WebTestClient` + `StepVerifier` as a matched pair: the former drives the HTTP
layer, the latter asserts on any reactive stream buried inside the response —
together they give you full confidence in both the contract and the actual
streaming behavior of your endpoints.
