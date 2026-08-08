# Filter Chain

## In Simple Terms

The filter chain is the line of registered `WebFilter`s a request passes
through on its way to your controller (and again on the way back, for the
response). Each filter decides whether to hand off to the next filter or
controller (via `chain.filter(exchange)`) — and can run logic both before
and after that call.

## Simple Example

```java
@Component
@Order(1)
public class FirstFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("First filter: before");
        return chain.filter(exchange)
            .doOnSuccess(v -> System.out.println("First filter: after"));
    }
}

@Component
@Order(2)
public class SecondFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("Second filter: before");
        return chain.filter(exchange)
            .doOnSuccess(v -> System.out.println("Second filter: after"));
    }
}
```

Execution order for a single request:

```
First filter: before
Second filter: before
--- Controller executes ---
Second filter: after
First filter: after
```

Notice the "after" part runs in *reverse* order — just like nested
function calls, the last filter to start "before" is the first to finish
"after."

## Why It Matters

Understanding this nested, ordered execution model matters for layering
concerns correctly — an authentication filter should usually run *before*
a logging filter that records who's authenticated, meaning the auth filter
needs a lower `@Order` value (it runs earlier).
