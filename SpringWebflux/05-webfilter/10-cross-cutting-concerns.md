# Cross-Cutting Concerns

## In Simple Terms

"Cross-cutting concerns" are bits of functionality that apply broadly
across most (or all) of an app — logging, security, metrics, header
checks — rather than being tied to any one feature. `WebFilter` is Spring
WebFlux's main tool for handling these in one central place, instead of
copying the same logic into every controller.

## Simple Example

A typical filter chain covering several cross-cutting concerns together:

```java
@Component @Order(1) public class AuthenticationFilter implements WebFilter { /* ... */ }
@Component @Order(2) public class AuthorizationFilter implements WebFilter { /* ... */ }
@Component @Order(3) public class RequestLoggingFilter implements WebFilter { /* ... */ }
@Component @Order(4) public class MetricsWebFilter implements WebFilter { /* ... */ }
```

Each filter handles exactly one concern, and they're composed together
through the filter chain — none of this logic needs to be repeated inside
individual `@RestController` classes.

## Why It Matters

Centralizing cross-cutting concerns in filters — instead of scattering
them across controllers — follows the broader idea of separation of
concerns: your controllers stay focused purely on business logic, while
filters handle the "plumbing" every request needs, no matter which
endpoint it's headed to.
