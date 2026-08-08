# Logging (WebFilter)

## In Simple Terms

A logging `WebFilter` records what's happening with every request and
response passing through your app — method, path, status code, duration —
all in one central place, instead of sprinkling logging statements across
every controller method individually.

## Simple Example

```java
@Component
public class RequestLoggingFilter implements WebFilter {

    private static final Logger log = LoggerFactory.getLogger(RequestLoggingFilter.class);

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        long start = System.currentTimeMillis();
        String method = exchange.getRequest().getMethod().name();
        String path = exchange.getRequest().getPath().value();

        return chain.filter(exchange)
            .doFinally(signalType -> {
                long duration = System.currentTimeMillis() - start;
                int status = exchange.getResponse().getStatusCode() != null
                    ? exchange.getResponse().getStatusCode().value() : 0;
                log.info("{} {} -> {} ({}ms)", method, path, status, duration);
            });
    }
}
```

Using `.doFinally()` instead of `.doOnSuccess()` makes sure logging still
happens even if the request fails or the client cancels partway through.

## Why It Matters

Centralized request logging through a `WebFilter` gives you consistent
visibility across your whole API — essential for debugging production
issues and keeping an eye on overall health — without scattering logging
calls throughout individual controllers.
