# Request Body

## In Simple Terms

`WebClient` offers two main ways to supply a request body: `.bodyValue(obj)` for a
value you already have, and `.body(publisher, Class)` for supplying the body
reactively (as a `Mono`/`Flux`) — useful for streaming request bodies or when the
body itself comes from an upstream asynchronous source.

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

Choosing the right body-supplying method keeps your `WebClient` calls fully
non-blocking and composable — `.body(mono, Class)` in particular lets you chain an
outgoing HTTP call directly onto an upstream asynchronous computation, without
needing to `.block()` to get a plain value first (which would defeat the purpose of
using WebClient).
