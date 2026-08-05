# Authentication

## In Simple Terms

Authentication answers the question "who is making this request?" A `WebFilter` is a
natural place to implement custom authentication logic — extracting credentials
(e.g., a bearer token) from the request, validating them, and attaching the
authenticated identity to the exchange for later use by your controllers.

## Simple Example

```java
@Component
public class TokenAuthenticationFilter implements WebFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String authHeader = exchange.getRequest().getHeaders().getFirst("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete(); // short-circuit, never reaches controller
        }

        String token = authHeader.substring(7);
        return validateToken(token)
            .flatMap(user -> {
                exchange.getAttributes().put("authenticatedUser", user);
                return chain.filter(exchange);
            })
            .switchIfEmpty(Mono.defer(() -> {
                exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                return exchange.getResponse().setComplete();
            }));
    }
}
```

(In real applications, Spring Security's reactive support is generally preferred over
hand-rolled filters like this for anything beyond simple demos.)

## Why It Matters

Implementing authentication as a `WebFilter` ensures **every** endpoint is protected
consistently, without needing to remember to add authentication checks manually in
each controller method — a single, centralized gatekeeper for the entire application.
