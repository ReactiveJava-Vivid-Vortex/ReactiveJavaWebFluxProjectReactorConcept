# Blocking vs Non-Blocking I/O

## In Simple Terms

**Blocking I/O**: when your code asks for data (a database row, a file, a network
response), the calling thread **freezes** until the data is ready — it can do
nothing else in the meantime.

**Non-blocking I/O**: the calling thread asks for data and immediately moves on to
other work. When the data becomes available, it's delivered via a callback/event,
without the thread ever having to sit idle waiting.

## Simple Example

```java
// Blocking: this thread does nothing else until the DB responds
User user = jdbcTemplate.queryForObject(sql, User.class); // FREEZES here

// Non-blocking: this returns instantly; the actual DB call happens
// in the background, and the result arrives later via the Mono
Mono<User> userMono = r2dbcTemplate.selectOne(query, User.class); // no freeze
```

Analogy: a blocking waiter takes an order and stands at the kitchen window until food
is ready before serving anyone else. A non-blocking waiter takes the order, moves on
to other tables immediately, and comes back to deliver food only once it's actually
ready (having been notified).

## Why It Matters

Spring WebFlux is built entirely around non-blocking I/O (via Netty). This is what
allows it to handle a huge number of concurrent connections using only a small
number of threads — every thread is always doing productive work, never frozen
waiting on a specific piece of I/O to complete.
