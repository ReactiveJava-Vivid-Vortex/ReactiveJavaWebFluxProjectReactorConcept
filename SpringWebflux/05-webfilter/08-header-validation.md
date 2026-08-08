# Header Validation

## In Simple Terms

A `WebFilter` can check that required HTTP headers (an API key, a client
version header, a content type) are present and correct before a request
even reaches a controller — rejecting bad or missing headers early and
consistently across your whole API.

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

Checking required headers in one central filter makes sure every endpoint
enforces the same rules — instead of each controller having to remember
to check for a header individually (and risking inconsistency if someone
forgets).
