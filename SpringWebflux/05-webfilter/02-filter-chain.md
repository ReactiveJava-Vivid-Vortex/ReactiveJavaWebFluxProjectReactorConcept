# Filter Chain

## In Simple Terms

The **filter chain** is the sequence of registered `WebFilter`s that a request passes
through before reaching your controller (and again on the way back, for the
response). Each filter decides whether to pass control to the next filter/controller
(via `chain.filter(exchange)`) — and can run logic both before and after that call.

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

Notice the "after" logic runs in **reverse order** — like nested function calls,
the last filter to run "before" is the first to finish "after."

## Why It Matters

Understanding the chain's nested, ordered execution model is essential for
correctly layering concerns — e.g., an authentication filter should typically run
*before* a logging filter that records the authenticated user, meaning the auth
filter needs a lower `@Order` value (runs earlier).
