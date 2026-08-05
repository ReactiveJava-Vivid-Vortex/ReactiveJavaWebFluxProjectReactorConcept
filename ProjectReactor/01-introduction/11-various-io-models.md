# Various I/O Models

## In Simple Terms

There isn't just one way to do I/O. Over the years, a few distinct models have
emerged, each with different trade-offs:

1. **Blocking I/O** — one thread per connection; thread waits (blocks) for each
   operation to complete. Simple to write, but doesn't scale well.
2. **Non-blocking I/O (polling)** — a thread repeatedly asks "is data ready yet?" for
   many connections in a loop, without waiting on any single one. Wastes CPU cycles on
   polling if not careful.
3. **I/O Multiplexing (select/poll/epoll)** — a thread asks the OS "tell me which of
   these many connections actually have data ready," and only then processes those.
   This is what modern non-blocking servers (like Netty, used by WebFlux) use.
4. **Asynchronous I/O (AIO)** — the OS itself performs the I/O operation in the
   background and notifies your application via a callback/completion event when it's
   fully done (used more on Windows, and in some newer Linux APIs like `io_uring`).

## Simple Example

```
Blocking:        Ask -> Wait (frozen) -> Get Data
Polling:         Ask -> "not ready" -> Ask again -> "not ready" -> ... -> Get Data
Multiplexing:    Watch 1000 sockets -> OS says "these 3 are ready" -> Read those 3
Async I/O:       Ask -> OS does it fully in background -> "Here's your data" (callback)
```

## Why It Matters for Reactive Programming

Spring WebFlux runs on **Netty**, which uses the I/O multiplexing model (via
`epoll`/`kqueue`/NIO Selectors). This is why WebFlux can handle huge numbers of
concurrent connections with only a small, fixed thread pool (often named
`reactor-http-nio-*`) — a handful of threads efficiently multiplex thousands of
sockets instead of dedicating one thread per connection.
