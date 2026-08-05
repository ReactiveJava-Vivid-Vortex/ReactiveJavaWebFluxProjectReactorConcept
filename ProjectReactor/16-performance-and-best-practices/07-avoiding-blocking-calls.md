# Avoiding Blocking Calls

## In Simple Terms

The final, most critical best practice: identify and eliminate (or properly isolate)
every blocking call in a reactive pipeline. Blocking calls hiding inside seemingly
innocent code are the most common cause of production reactive performance problems.

## Common Sources of Hidden Blocking Calls

```java
// 1. Legacy JDBC / blocking database drivers
Mono.fromCallable(() -> jdbcTemplate.queryForObject(sql, User.class));

// 2. Blocking HTTP clients (e.g., old RestTemplate instead of WebClient)
Mono.fromCallable(() -> restTemplate.getForObject(url, Response.class));

// 3. Thread.sleep() anywhere in a reactive pipeline
Mono.fromCallable(() -> { Thread.sleep(100); return "value"; });

// 4. Synchronized blocks / locks that can be held for a while
Mono.fromCallable(() -> {
    synchronized (lock) {
        return expensiveSynchronizedOperation();
    }
});

// 5. Blocking file I/O (java.io.* instead of async alternatives)
Mono.fromCallable(() -> Files.readString(path));
```

## The Fix

For any unavoidable blocking call, always isolate it:

```java
Mono.fromCallable(() -> jdbcTemplate.queryForObject(sql, User.class))
    .subscribeOn(Schedulers.boundedElastic()); // isolate on a dedicated pool
```

Better yet, replace it with a genuinely non-blocking alternative where one exists
(R2DBC instead of JDBC, WebClient instead of RestTemplate).

## Why It Matters

A single blocking call left running on an event-loop thread can silently degrade the
performance of an entire application under load — often invisible in low-traffic
testing, but catastrophic once real concurrent load hits production. Systematically
auditing for hidden blocking calls (and isolating or eliminating them) is one of the
highest-value activities in maintaining a healthy reactive system.
