# Passing Information Between Filters

## In Simple Terms

Filters often need to share data with each other (and with the eventual controller)
— e.g., an authentication filter determining the current user, which a later
authorization filter and the controller both need to access. `ServerWebExchange`
provides an **attributes map** specifically for this purpose.

## Simple Example

```java
@Component
@Order(1)
public class UserContextFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        User user = resolveUserFromToken(exchange);
        exchange.getAttributes().put("currentUser", user); // store for later use
        return chain.filter(exchange);
    }
}
```

Reading the attribute later — in another filter, or in a controller:

```java
@GetMapping("/profile")
public Mono<ProfileDto> getProfile(ServerWebExchange exchange) {
    User user = exchange.getAttribute("currentUser");
    return profileService.getProfile(user.getId());
}
```

**Important gotcha:** avoid using `ThreadLocal` for this purpose in reactive code —
since a request's processing may hop across multiple threads
(see [[thread-affinity]] in the ProjectReactor notes), `ThreadLocal` values set in
one filter may not be visible later in the chain. `ServerWebExchange` attributes (or
Reactor's `Context`) are the reactive-safe alternative.

## Why It Matters

Understanding this attribute-passing mechanism avoids a very common reactive
mistake: trying to use `ThreadLocal` (which works in traditional Spring MVC, but
breaks unpredictably in WebFlux due to thread-hopping) to share request-scoped data
between filters and controllers.
