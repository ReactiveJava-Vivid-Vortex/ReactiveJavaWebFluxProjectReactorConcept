# WebFilter — Topic Overview

## What Is This Topic About? (In Simple Terms)

Some concerns apply to almost **every** request in your application — logging,
authentication, metrics, header validation — and you don't want to repeat that logic
in every single controller method. `WebFilter` is WebFlux's answer: a reactive
version of the Servlet `Filter`, letting you intercept every request/response
centrally, before and after it reaches your controller.

```java
@Component
public class LoggingWebFilter implements WebFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("Incoming: " + exchange.getRequest().getPath());
        return chain.filter(exchange) // pass control onward
            .doOnSuccess(v -> System.out.println("Completed"));
    }
}
```

When you have multiple filters, their **order** matters — controlled via `@Order`
(lower runs first). A common layering: authenticate first, authorize second, then
log/monitor. Execution nests like function calls: the "before" logic runs in
order, but the "after" logic (anything chained after `chain.filter(exchange)`) runs
in **reverse** order.

Filters often need to share data — an authentication filter determining "who is
this user," which a later authorization filter and the controller both need. Use
`exchange.getAttributes()` for this, **not** `ThreadLocal` — because a reactive
request's processing can hop across multiple threads, `ThreadLocal` values set in
one filter may simply not be visible later in the chain.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **What is WebFilter** | Reactive equivalent of Servlet `Filter` — intercepts every request/response; `filter()` returns `Mono<Void>`. |
| 2 | **Filter Chain** | The nested sequence of filters a request passes through; "after" logic runs in reverse order. |
| 3 | **Filter Ordering** | Control execution order with `@Order` (lower runs first) — e.g., authenticate before authorize. |
| 4 | **Authentication** | "Who is making this request?" — extract/validate credentials, attach identity to the exchange. |
| 5 | **Authorization** | "Is this user allowed to do this?" — separate concern from authentication, usually runs right after it. |
| 6 | **Logging** | Centralized request/response logging (method, path, status, duration) in one place. |
| 7 | **Monitoring** | Feed metrics (via Micrometer) into a monitoring system for every request, automatically. |
| 8 | **Header Validation** | Reject requests missing required headers (API key, etc.) before they reach a controller. |
| 9 | **Passing information between filters** | Use `exchange.getAttributes()`, NOT `ThreadLocal` — requests can hop across threads. |
| 10 | **Cross-cutting concerns** | The umbrella term for everything above — logic that applies broadly, centralized instead of duplicated. |

## How It All Fits Together

```
Request arrives
      │
      ▼
Filter 1 (@Order 1, e.g. Authentication) — "before" logic
      │         stores user info via exchange.getAttributes()
      ▼
Filter 2 (@Order 2, e.g. Authorization) — reads attribute, checks permission
      │
      ▼
Filter 3 (@Order 3, e.g. Logging/Metrics)
      │
      ▼
Controller executes
      │
      ▼ (unwinding back UP through the chain, in REVERSE order)
Filter 3 "after" logic → Filter 2 "after" logic → Filter 1 "after" logic
      │
      ▼
Response sent to client
```

The pattern to remember: **filters = centralized plumbing that every request goes
through, so controllers stay focused purely on their own business logic.**
