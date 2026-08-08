# Authorization

## In Simple Terms

Authorization answers "is this person allowed to do this?" — a separate
question from authentication ("who are you?"). A filter can check the
authenticated user's roles or permissions (usually set by an earlier
authentication filter) against what a given resource requires.

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

Keeping authorization separate from authentication (as its own filter,
ordered correctly) keeps each concern focused and easy to test on its
own — you can change "who's allowed into `/admin`" without touching how
users get authenticated in the first place, and vice versa.
