# Filter Ordering

## In Simple Terms

When you have several `WebFilter`s, the order they run in matters — you
control it with `@Order` (lower numbers run first) or by implementing
`Ordered`. Getting the order right makes sure dependent steps (like
"authenticate before authorizing," or "authenticate before logging who's
authenticated") happen in the right sequence.

## Simple Example

```java
@Component
@Order(1) // runs first
public class AuthenticationFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        // authenticate, then store user info as an exchange attribute
        exchange.getAttributes().put("user", authenticate(exchange));
        return chain.filter(exchange);
    }
}

@Component
@Order(2) // runs second - can safely rely on "user" attribute already being set
public class AuditLoggingFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        Object user = exchange.getAttribute("user");
        System.out.println("Request by: " + user);
        return chain.filter(exchange);
    }
}
```

Alternative: implementing `Ordered` directly instead of using the
annotation:

```java
@Component
public class AuthenticationFilter implements WebFilter, Ordered {
    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE; // guarantees it runs first
    }
    // ... filter() implementation
}
```

## Why It Matters

Getting filter order wrong is a sneaky source of bugs — like a logging
filter trying to read authenticated user info that hasn't been set yet
because the auth filter actually runs *after* it. Being deliberate about
`@Order` values avoids these ordering-dependent failures.
