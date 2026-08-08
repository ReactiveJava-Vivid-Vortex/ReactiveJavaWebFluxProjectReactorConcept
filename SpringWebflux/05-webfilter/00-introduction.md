# Q1. What Is a WebFilter?

## Simple Explanation (Think of Airport Security Before Every Gate)

Before any passenger (request) reaches their gate (controller), they pass through
a series of checkpoints — ID check, bag scan, boarding pass scan — each run by a
different station, in a fixed order, applying to **every** passenger the same way.
`WebFilter` is that checkpoint system for WebFlux requests.

```java
@Component
public class LoggingWebFilter implements WebFilter {
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("Incoming: " + exchange.getRequest().getPath());
        return chain.filter(exchange) // pass to the NEXT checkpoint (or the gate itself)
            .doOnSuccess(v -> System.out.println("Completed"));
    }
}
```

---

## Q2. How Does the Filter Chain Actually Execute?

```java
@Component @Order(1) public class FirstFilter implements WebFilter {
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("First: before");
        return chain.filter(exchange).doOnSuccess(v -> System.out.println("First: after"));
    }
}
@Component @Order(2) public class SecondFilter implements WebFilter {
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("Second: before");
        return chain.filter(exchange).doOnSuccess(v -> System.out.println("Second: after"));
    }
}
```

```
First: before
Second: before
--- Controller executes ---
Second: after      <- "after" logic runs in REVERSE order, like nested calls
First: after
```

---

## Q3. Why Does Filter Order Matter So Much?

```java
@Component @Order(1) // MUST run first
public class AuthenticationFilter implements WebFilter {
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        exchange.getAttributes().put("user", authenticate(exchange));
        return chain.filter(exchange);
    }
}

@Component @Order(2) // safely relies on "user" already being set
public class AuditLoggingFilter implements WebFilter {
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        System.out.println("Request by: " + exchange.getAttribute("user"));
        return chain.filter(exchange);
    }
}
```

If these were reversed, `AuditLoggingFilter` would read `null` for "user" — a
classic ordering bug.

---

## Q4. How Do Filters Share Data With Each Other and the Controller?

```java
// WRONG in reactive code — a request can hop across threads mid-pipeline
private static final ThreadLocal<User> currentUser = new ThreadLocal<>();

// CORRECT — use the exchange's attribute map
exchange.getAttributes().put("currentUser", user);

// later, anywhere downstream (another filter, or the controller):
User user = exchange.getAttribute("currentUser");
```

Because reactive request processing can cross threads (see Thread Affinity in the
Project Reactor Threading topic), `ThreadLocal` values set in one filter can
simply vanish by the time a later filter runs.

---

## Q5. What Are Filters Typically Used For?

| Use Case | Example |
|---|---|
| Authentication | Extract/validate a token, attach identity to the exchange |
| Authorization | Check the attached identity's permissions for this route |
| Logging | Record method, path, status, duration for every request |
| Monitoring | Feed request metrics into Micrometer/Prometheus |
| Header validation | Reject requests missing a required API key header |

```java
@Component
public class RequestLoggingFilter implements WebFilter {
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        long start = System.currentTimeMillis();
        return chain.filter(exchange)
            .doFinally(signal -> { // runs even on error/cancellation — see doFinally() in Reactor Operators
                long duration = System.currentTimeMillis() - start;
                log.info("{} -> {}ms", exchange.getRequest().getPath(), duration);
            });
    }
}
```

---

## Q6. Interview-Style Q&A

### Can I use `ThreadLocal` to pass data between filters?

**No, don't.** Use `exchange.getAttributes()` — reactive request processing can
hop across multiple threads, so `ThreadLocal` isn't reliable.

### If I have Authentication at `@Order(2)` and Authorization at `@Order(1)`, what breaks?

Authorization would run **before** authentication has set the user's identity,
likely resulting in every request being denied (or incorrectly allowed).

### Does `chain.filter(exchange)` block until the controller finishes?

No — it returns a `Mono<Void>` representing "the rest of the chain finishing,"
composed reactively, not a blocking call.

---

## Q7. Summary

```
Request arrives
      │
      ▼
Filter 1 (@Order 1, e.g. Auth) — "before" logic, stores user via getAttributes()
      │
      ▼
Filter 2 (@Order 2, e.g. Logging) — "before" logic
      │
      ▼
Controller executes
      │
      ▼ (unwinds in REVERSE order)
Filter 2 "after" → Filter 1 "after"
      │
      ▼
Response sent
```

### One sentence to remember

> **"WebFilter is a chain of security checkpoints every request passes through
> — use @Order to sequence them correctly, and exchange attributes (never
> ThreadLocal) to pass data between them."**
