# Cross-Cutting Concerns

## In Simple Terms

"Cross-cutting concerns" are pieces of functionality that apply broadly across many
(or all) parts of an application — logging, security, metrics, header validation —
rather than being specific to any one feature. `WebFilter` is Spring WebFlux's
primary mechanism for implementing these concerns in one centralized place, instead
of duplicating the same logic across every controller.

## Simple Example

A typical filter chain addressing multiple cross-cutting concerns together:

```java
@Component @Order(1) public class AuthenticationFilter implements WebFilter { /* ... */ }
@Component @Order(2) public class AuthorizationFilter implements WebFilter { /* ... */ }
@Component @Order(3) public class RequestLoggingFilter implements WebFilter { /* ... */ }
@Component @Order(4) public class MetricsWebFilter implements WebFilter { /* ... */ }
```

Each filter addresses exactly one concern, composed together via the filter chain —
none of this logic needs to be duplicated inside individual `@RestController`
classes.

## Why It Matters

Centralizing cross-cutting concerns in filters (rather than scattering them across
controllers) follows the broader software engineering principle of **separation of
concerns** — your controllers stay focused purely on their specific business logic,
while filters uniformly handle the "plumbing" that every request needs, regardless
of which endpoint it targets.
