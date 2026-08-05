# Monitoring

## In Simple Terms

Beyond simple logging, a `WebFilter` can also feed metrics into a monitoring system
(like Micrometer, feeding into Prometheus/Grafana) — tracking request counts,
latencies, and error rates per endpoint, giving you visibility into your
application's health and performance in production.

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

Centralized monitoring via a `WebFilter` gives you application-wide observability
"for free" — every endpoint automatically reports metrics, letting you build
dashboards and alerts (e.g., "alert if error rate on `/checkout` exceeds 5%") without
needing to instrument each controller method by hand.
