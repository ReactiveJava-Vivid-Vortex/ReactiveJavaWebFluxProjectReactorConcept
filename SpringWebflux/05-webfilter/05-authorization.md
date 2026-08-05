# Authorization

## In Simple Terms

Authorization answers "is this authenticated user *allowed* to do this?" — a
separate concern from authentication ("who are you?"). A filter can check the
authenticated user's roles/permissions (usually set by an earlier authentication
filter) against what the requested resource requires.

## Simple Example

```java
@Component
@Order(2) // runs AFTER authentication
public class AuthorizationFilter implements WebFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String path = exchange.getRequest().getPath().value();

        if (path.startsWith("/admin")) {
            User user = exchange.getAttribute("authenticatedUser");
            if (user == null || !user.getRoles().contains("ADMIN")) {
                exchange.getResponse().setStatusCode(HttpStatus.FORBIDDEN);
                return exchange.getResponse().setComplete();
            }
        }

        return chain.filter(exchange);
    }
}
```

## Why It Matters

Separating authorization from authentication (as distinct filters, ordered
correctly) keeps each concern focused and independently testable — you can change
"who can access `/admin`" logic without touching how users are authenticated in the
first place, and vice versa.
