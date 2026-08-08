# Various I/O Models

## In Simple Terms

There's more than one way to handle I/O. Over the years, a few different
approaches have shown up, each with its own trade-offs:

1. **Blocking I/O** — one thread per connection, and it just waits for each
   operation. Easy to write, but doesn't scale well.
2. **Polling** — a thread keeps asking "is it ready yet?" over and over, for many
   connections, in a loop. Wastes CPU time if done carelessly.
3. **I/O Multiplexing** — a thread asks the OS "which of these connections
   actually have data ready right now?" and only deals with those. This is what
   modern servers like Netty (used by WebFlux) use.
4. **Async I/O** — the OS itself does the work in the background and taps you on
   the shoulder with a callback once it's fully done (more common on Windows, and
   in newer Linux APIs like `io_uring`).

## Simple Example

```
Blocking:        Ask -> Wait (frozen) -> Get Data
Polling:         Ask -> "not ready" -> Ask again -> "not ready" -> ... -> Get Data
Multiplexing:    Watch 1000 connections -> OS says "these 3 are ready" -> Read those 3
Async I/O:       Ask -> OS does it fully in the background -> "Here's your data" (callback)
```

## Why It Matters for Reactive Programming

Spring WebFlux runs on **Netty**, which uses the multiplexing approach (option
3). That's why WebFlux can handle huge numbers of connections with just a small,
fixed group of threads — a few threads efficiently watch thousands of
connections, instead of tying up one thread per connection.
