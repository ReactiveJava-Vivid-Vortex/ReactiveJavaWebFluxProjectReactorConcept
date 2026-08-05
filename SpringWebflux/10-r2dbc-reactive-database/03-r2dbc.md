# R2DBC

## In Simple Terms

**R2DBC** (Reactive Relational Database Connectivity) is a specification (and set of
drivers) providing truly non-blocking access to relational databases (PostgreSQL,
MySQL, SQL Server, H2, etc.) — the reactive counterpart to JDBC, which is
fundamentally blocking by design and can't be made non-blocking no matter how it's
wrapped.

## Simple Example

Configuring an R2DBC connection (PostgreSQL example):

```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/mydb
    username: myuser
    password: mypassword
```

Required dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>
```

With this configured, `ReactiveCrudRepository` and `R2dbcEntityTemplate` beans become
available for fully non-blocking database access.

## Why It Matters

R2DBC exists specifically because JDBC's blocking nature is fundamentally
incompatible with a truly non-blocking application — no amount of wrapping a JDBC
call in a `Mono` (other than isolating it on `boundedElastic()`, which still uses a
thread per blocking call) achieves the same efficiency as a genuinely non-blocking
database driver built from the ground up around asynchronous I/O.
