# WebTestClient

## In Simple Terms

`WebTestClient` is Spring's dedicated testing tool for WebFlux applications — it
lets you send real (or mocked) HTTP requests to your application and assert on the
response, similar in spirit to `MockMvc` for traditional Spring MVC, but built for
the reactive, non-blocking programming model.

## Simple Example

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ProductControllerTest {

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

    @Test
    void getProduct_notFound_returns404() {
        webTestClient.get()
            .uri("/products/{id}", "nonexistent")
            .exchange()
            .expectStatus().isNotFound();
    }
}
```

## Why It Matters

`WebTestClient` handles the reactive complexity for you — internally, it subscribes
to your controller's `Mono`/`Flux` response and blocks in a controlled,
test-appropriate way until the result is available, letting you write straightforward
assertions without manually dealing with subscriptions or `StepVerifier` at the HTTP
layer.
