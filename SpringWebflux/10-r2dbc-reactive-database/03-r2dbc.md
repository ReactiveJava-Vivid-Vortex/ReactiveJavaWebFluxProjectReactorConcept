# R2DBC

## In Simple Terms

R2DBC (Reactive Relational Database Connectivity) is a spec and set of
drivers for truly non-blocking access to relational databases (PostgreSQL,
MySQL, SQL Server, H2, and more) — the reactive counterpart to JDBC, which
is blocking by nature and can't be turned non-blocking no matter how you
wrap it.

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

With this set up, `ReactiveCrudRepository` and `R2dbcEntityTemplate` beans
become available for fully non-blocking database access.

## Why It Matters

R2DBC exists specifically because JDBC's blocking nature just doesn't fit
with a genuinely non-blocking app — wrapping a JDBC call in a `Mono`
(other than isolating it on `boundedElastic()`, which still burns one
thread per blocking call) can't match the efficiency of a database driver
actually built from scratch around async I/O.
