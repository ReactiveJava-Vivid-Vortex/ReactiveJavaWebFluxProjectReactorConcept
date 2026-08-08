# Request Body

## In Simple Terms

`WebClient` gives you two main ways to send a request body:
`.bodyValue(obj)` for a value you already have in hand, and
`.body(publisher, Class)` for supplying the body reactively (as a
`Mono`/`Flux`) — useful for streaming request bodies or when the body
itself comes from an upstream async source.

## Simple Example

```java
// bodyValue() - simplest case, you already have the object
webClient.post()
    .uri("/orders")
    .bodyValue(new OrderRequest("PROD-1", 5))
    .retrieve()
    .bodyToMono(OrderResponse.class);

// body() with a Mono - the value comes from an upstream reactive source
Mono<OrderRequest> orderRequestMono = validateAndBuildOrder(rawInput);

webClient.post()
    .uri("/orders")
    .body(orderRequestMono, OrderRequest.class)
    .retrieve()
    .bodyToMono(OrderResponse.class);

// body() with a Flux - streaming multiple items as the request body (e.g. NDJSON)
Flux<OrderRequest> orderStream = getOrdersToSubmit();

webClient.post()
    .uri("/orders/batch")
    .contentType(MediaType.APPLICATION_NDJSON)
    .body(orderStream, OrderRequest.class)
    .retrieve()
    .bodyToMono(BatchResult.class);
```

## Why It Matters

Picking the right body-supplying method keeps `WebClient` calls fully
non-blocking and easy to compose — `.body(mono, Class)` in particular lets
you chain an outgoing call directly onto an upstream async computation,
without needing to `.block()` first to get a plain value (which would
defeat the whole point of using `WebClient`).
