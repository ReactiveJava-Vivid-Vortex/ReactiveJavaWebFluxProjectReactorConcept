# Blocking vs Non-Blocking I/O

## In Simple Terms

**Blocking I/O**: your code asks for some data (a database row, a file, a
network response), and the thread that asked just freezes right there until
the data shows up — it can't do anything else in the meantime.

**Non-blocking I/O**: the thread asks for data and immediately moves on to
other work. When the data's finally ready, it gets handed over through a
callback or event — the thread never had to just sit there waiting.

## Simple Example

```java
// Blocking: this thread does nothing else until the DB responds
User user = jdbcTemplate.queryForObject(sql, User.class); // FREEZES here

// Non-blocking: this returns instantly; the actual DB call happens
// in the background, and the result arrives later via the Mono
Mono<User> userMono = r2dbcTemplate.selectOne(query, User.class); // no freeze
```

Think of it like two waiters: a blocking waiter takes an order and stands
at the kitchen window until the food's ready before serving anyone else. A
non-blocking waiter takes the order, immediately moves on to other tables,
and only comes back to deliver the food once it's actually ready (someone
lets them know).

## Why It Matters

Spring WebFlux is built entirely around non-blocking I/O (through Netty).
That's exactly what lets it handle a huge number of connections at once
using only a handful of threads — every thread stays busy doing real work,
never frozen waiting on one specific piece of I/O.
