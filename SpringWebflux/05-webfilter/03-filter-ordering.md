# Filter Ordering

## In Simple Terms

When you have multiple `WebFilter`s, their execution order matters — you control it
using the `@Order` annotation (lower values run first) or by implementing
`Ordered`. Getting the order right ensures dependent concerns (like "authenticate
before authorizing," or "authenticate before logging the authenticated user") happen
in the correct sequence.

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

Alternative: implementing `Ordered` directly instead of using the annotation:

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

Incorrect filter ordering is a subtle source of bugs — e.g., a logging filter trying
to read authenticated user info that hasn't been set yet because the authentication
filter runs *after* it. Being deliberate about `@Order` values prevents these
ordering-dependent failures.
