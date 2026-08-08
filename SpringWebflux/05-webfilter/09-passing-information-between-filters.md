# Passing Information Between Filters

## In Simple Terms

Filters often need to share data with each other (and with the eventual
controller) — an authentication filter figuring out the current user,
which a later authorization filter and the controller both need to see.
`ServerWebExchange` provides an attributes map made exactly for this.

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

**Watch out for this:** avoid using `ThreadLocal` for this in reactive
code — since one request's processing can hop across several threads
(see [[thread-affinity]] in the Project Reactor notes), a `ThreadLocal`
value set in one filter might not even be visible later in the chain.
`ServerWebExchange` attributes (or Reactor's `Context`) are the safe way to
do this reactively.

## Why It Matters

Knowing about this attribute-passing mechanism helps you avoid a really
common reactive mistake: reaching for `ThreadLocal` (which works fine in
traditional Spring MVC, but breaks unpredictably in WebFlux due to
thread-hopping) to share request-scoped data between filters and
controllers.
