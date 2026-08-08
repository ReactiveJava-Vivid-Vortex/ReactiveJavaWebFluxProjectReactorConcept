# Q1. How Do I Test a WebFlux Endpoint End-to-End?

## Simple Explanation (Think of a Mystery Shopper vs Inspecting Kitchen Equipment)

A unit test is like inspecting the oven in isolation — does it heat correctly?
An integration test is a **mystery shopper**: walk in the front door, order food,
and check what actually comes out — verifying the controller, service,
repository, and (ideally) a real database all work together correctly.

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

`WebTestClient` handles all the reactive complexity internally — subscribing to
your controller's `Mono`/`Flux` and blocking in a test-appropriate, controlled way
— you never manage subscriptions manually.

---

## Q2. How Do I Test a Streaming (NDJSON/SSE) Endpoint?

A streaming endpoint's response IS a `Flux`, not one object — so you need
`StepVerifier` on top of `WebTestClient` for the actual body assertions:

```java
FluxExchangeResult<ProductDto> result = webTestClient.get()
    .uri("/products/stream").accept(MediaType.APPLICATION_NDJSON)
    .exchange().expectStatus().isOk().returnResult(ProductDto.class);

StepVerifier.create(result.getResponseBody()).expectNextCount(3).verifyComplete();
```

---

## Q3. How Do I Test Against a Real Database Instead of Mocks?

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWebTestClient
@Testcontainers
class ProductIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15");

    @DynamicPropertySource
    static void configureR2dbc(DynamicPropertyRegistry registry) {
        registry.add("spring.r2dbc.url", () ->
            "r2dbc:postgresql://" + postgres.getHost() + ":" + postgres.getFirstMappedPort() + "/test");
    }
    // ... test methods
}
```

This catches issues mocked unit tests miss entirely: incorrect SQL, serialization
mismatches, misconfigured routes.

---

## Q4. Interview-Style Q&A

### Does `WebTestClient` require a running server?

It can work either way — bound to a running server (`RANDOM_PORT`) for true
integration tests, or bound directly to your `RouterFunction`/controllers in
isolation for faster, more focused tests.

### Why can't I just call `.block()` on the controller's returned Mono and assert normally?

You *could* in principle for a plain unit test of a service method, but for
testing the full HTTP layer (status codes, headers, serialization), `WebTestClient`
is purpose-built and far more expressive.

### How do I assert a streaming endpoint never completes (e.g., an SSE feed)?

Use `StepVerifier` with `.thenCancel()` instead of `.verifyComplete()` — waiting
for completion on a genuinely infinite stream would hang the test forever.

---

## Q5. Summary

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

### One sentence to remember

> **"WebTestClient is a mystery shopper for your API — it drives the HTTP
> layer end-to-end, and StepVerifier steps in whenever the response itself is
> a stream, not a single object."**
