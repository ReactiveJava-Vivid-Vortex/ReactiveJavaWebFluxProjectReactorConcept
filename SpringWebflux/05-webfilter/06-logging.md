# Logging (WebFilter)

## In Simple Terms

A logging `WebFilter` records details about every request/response passing through
your application — method, path, status code, and duration — in one centralized
place, rather than adding logging statements to every controller method individually.

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

Using `.doFinally()` (rather than `.doOnSuccess()`) ensures logging happens even if
the request fails with an error or is cancelled by the client.

## Why It Matters

Centralized request logging via `WebFilter` gives you consistent observability
across your entire API surface — essential for debugging production issues and
monitoring overall application health — without scattering logging calls throughout
individual controller methods.
