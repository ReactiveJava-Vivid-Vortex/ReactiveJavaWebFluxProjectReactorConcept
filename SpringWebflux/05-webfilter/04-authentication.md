# Authentication

## In Simple Terms

Authentication answers "who's making this request?" A `WebFilter` is a
natural place to put custom authentication logic — pulling credentials
(like a bearer token) out of the request, checking they're valid, and
attaching the identified user to the exchange so controllers can use it
later.

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

(In real projects, Spring Security's reactive support is usually the
better choice over a hand-rolled filter like this, beyond simple demos.)

## Why It Matters

Doing authentication as a `WebFilter` makes sure every endpoint is
protected the same way, without needing to remember to add checks
manually in each controller — one single, centralized gatekeeper for the
whole app.
