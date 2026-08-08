# What is WebFilter

## In Simple Terms

`WebFilter` is Spring WebFlux's version of the traditional Servlet
`Filter` — it lets you intercept every incoming request (and outgoing
response) before and after it reaches your controller, for things like
logging, authentication, or adding headers. Unlike Servlet filters,
`WebFilter` is fully reactive — its `filter()` method hands back a
`Mono<Void>`.

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

Just defining this as a `@Component` registers it automatically — every
request to your app now passes through this filter first.

## Why It Matters

`WebFilter` is the standard place to put logic that should apply
everywhere across your app, not just one controller — logging, security,
tweaking requests/responses — all written reactively, in step with the
rest of the non-blocking request flow.
