# Avoiding Blocking Calls

## In Simple Terms

The last, and most critical, best practice: find and remove (or properly
isolate) every blocking call hiding in a reactive pipeline. Blocking calls
hiding inside otherwise innocent-looking code are the most common cause of
reactive performance problems in production.

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

For any blocking call you genuinely can't avoid, always isolate it:

```java
Mono.fromCallable(() -> jdbcTemplate.queryForObject(sql, User.class))
    .subscribeOn(Schedulers.boundedElastic()); // isolate on a dedicated pool
```

Even better, swap it out for a truly non-blocking alternative where one
exists (R2DBC instead of JDBC, WebClient instead of RestTemplate).

## Why It Matters

A single blocking call left running on an event-loop thread can quietly
drag down the performance of an entire app under load — often invisible
during light testing, but a real problem once genuine concurrent traffic
hits it. Regularly hunting for hidden blocking calls (and isolating or
removing them) is one of the highest-value things you can do to keep a
reactive system healthy.
