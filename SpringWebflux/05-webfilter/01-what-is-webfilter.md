# What is WebFilter

## In Simple Terms

`WebFilter` is Spring WebFlux's equivalent of the traditional Servlet `Filter` — it
lets you intercept every incoming request (and outgoing response) **before** and
**after** it reaches your controller, for cross-cutting concerns like logging,
authentication, or adding headers. Unlike Servlet filters, `WebFilter` is fully
reactive — its `filter()` method returns a `Mono<Void>`.

## Simple Example

```java
@Component
public class LoggingWebFilter implements WebFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        String path = exchange.getRequest().getPath().value();
        System.out.println("Incoming request: " + path);

        return chain.filter(exchange) // pass control to the next filter/controller
            .doOnSuccess(v -> System.out.println("Completed: " + path));
    }
}
```

Simply defining this as a `@Component` registers it automatically — every request to
your WebFlux application now passes through this filter first.

## Why It Matters

`WebFilter` is the standard extension point for cross-cutting concerns that should
apply uniformly across your entire application (not just one controller) — logging,
security, request/response modification — all expressed reactively, consistent with
the rest of the non-blocking request pipeline.
