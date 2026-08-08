# Monitoring

## In Simple Terms

Beyond plain logging, a `WebFilter` can also feed metrics into a
monitoring system (like Micrometer, hooked up to Prometheus/Grafana) —
tracking request counts, response times, and error rates per endpoint,
giving you a picture of how your app is doing in production.

## Simple Example

```java
@Component
public class MetricsWebFilter implements WebFilter {

    private final MeterRegistry meterRegistry;

    public MetricsWebFilter(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        Timer.Sample sample = Timer.start(meterRegistry);
        String path = exchange.getRequest().getPath().value();

        return chain.filter(exchange)
            .doFinally(signalType -> {
                int status = exchange.getResponse().getStatusCode() != null
                    ? exchange.getResponse().getStatusCode().value() : 0;

                sample.stop(Timer.builder("http.requests")
                    .tag("path", path)
                    .tag("status", String.valueOf(status))
                    .register(meterRegistry));
            });
    }
}
```

## Why It Matters

Centralized monitoring through a `WebFilter` gives you app-wide
visibility "for free" — every endpoint automatically reports metrics,
letting you build dashboards and alerts (like "alert if the error rate on
`/checkout` goes above 5%") without instrumenting each controller method
by hand.
