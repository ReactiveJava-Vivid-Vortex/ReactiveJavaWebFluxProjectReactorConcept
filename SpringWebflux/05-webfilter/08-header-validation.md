# Header Validation

## In Simple Terms

A `WebFilter` can validate required HTTP headers (like an API key, a client version
header, or a content-type) **before** any request reaches a controller — rejecting
malformed or missing headers early and consistently across your entire API.

## Simple Example

```java
@Component
public class ApiKeyValidationFilter implements WebFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String apiKey = exchange.getRequest().getHeaders().getFirst("X-API-Key");

        if (apiKey == null || apiKey.isBlank()) {
            exchange.getResponse().setStatusCode(HttpStatus.BAD_REQUEST);
            DataBufferFactory bufferFactory = exchange.getResponse().bufferFactory();
            DataBuffer buffer = bufferFactory.wrap(
                "{\"error\":\"Missing X-API-Key header\"}".getBytes(StandardCharsets.UTF_8)
            );
            return exchange.getResponse().writeWith(Mono.just(buffer));
        }

        return chain.filter(exchange);
    }
}
```

## Why It Matters

Validating required headers centrally, in a filter, ensures every endpoint enforces
the same rules consistently — instead of each controller needing to remember to
check for a header individually (and risking inconsistency if one is forgotten).
