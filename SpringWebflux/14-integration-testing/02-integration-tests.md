# Integration Tests

## In Simple Terms

An "integration test" checks that several layers of your app (controller,
service, repository, and often a real or embedded database) actually work
together correctly — unlike a unit test, which tests one class on its own
with mocked dependencies.

## Simple Example

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

    @Autowired
    private WebTestClient webTestClient;

    @Test
    void createAndRetrieveProduct_fullFlow() {
        ProductDto newProduct = new ProductDto(null, "Widget", 9.99);

        // Create
        ProductDto created = webTestClient.post()
            .uri("/products")
            .bodyValue(newProduct)
            .exchange()
            .expectStatus().isCreated()
            .expectBody(ProductDto.class)
            .returnResult().getResponseBody();

        // Retrieve - verifying it actually persisted to the real database
        webTestClient.get()
            .uri("/products/{id}", created.id())
            .exchange()
            .expectStatus().isOk()
            .expectBody(ProductDto.class)
            .value(product -> assertThat(product.name()).isEqualTo("Widget"));
    }
}
```

## Why It Matters

Integration tests catch things unit tests (with mocked dependencies) just
can't see — wrong SQL queries, mismatched serialization, misconfigured
routes — by actually exercising the real interaction between your app's
layers, ideally against a real (or realistic, containerized) database.
