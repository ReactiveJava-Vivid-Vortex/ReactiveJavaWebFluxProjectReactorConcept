# Introduction — Topic Overview

## What Is This Topic About? (In Simple Terms)

Before you can appreciate why Project Reactor or Spring WebFlux exist, you need to
understand one basic truth about computers: **doing nothing while waiting is
expensive.**

A CPU can only run one instruction stream per core at a time. The operating system
juggles many programs by rapidly switching a core between different threads
(**CPU scheduling**), and each switch has overhead (**context switching**). Every
thread also costs real memory just to exist (**RAM/stack usage**).

Now think about a normal web server: for every request, it typically hands the work
to one thread, and that thread often has to **wait** — for a database to respond,
for a file to be read, for another service to answer. In the traditional
(**synchronous, blocking**) model, that thread just freezes during the wait, doing
nothing useful, but still consuming memory and being tracked by the OS scheduler.

**Asynchronous** and **non-blocking** programming flip this: instead of freezing,
the thread is released the instant it would have to wait, and gets notified later
(via a callback/event) when the result is ready. Multiply this by thousands of
concurrent requests, and the savings become enormous — this is precisely **why
reactive programming exists**.

**Simple analogy:** A blocking waiter takes your order, walks it to the kitchen, and
stands there until food is ready before serving anyone else. A non-blocking waiter
drops off the order and immediately serves other tables, coming back only when the
food is actually ready. The second waiter serves far more tables with the same
staff.

```java
// Blocking: the thread freezes here until the DB responds
User user = jdbcTemplate.queryForObject(sql, User.class);

// Non-blocking: returns immediately; execution resumes later, on data arrival
Mono<User> userMono = r2dbcTemplate.selectOne(query, User.class);
```

Reactive programming isn't automatically faster per-operation — it's about using a
**small number of threads far more efficiently** under high, I/O-heavy concurrency.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---------|-------------------|
| 1 | **Course overview** | The roadmap: understand hardware/OS fundamentals first, then blocking vs non-blocking, then why reactive programming exists. |
| 2 | **Process vs Thread** | A process = isolated program with its own memory; a thread = a worker inside a process, sharing memory with other threads in it. |
| 3 | **CPU scheduling** | The OS gives each thread a small time-slice on a CPU core, rapidly switching between threads to create the illusion of parallelism. |
| 4 | **RAM and memory model** | Each thread needs its own stack (~1MB default) + shares the process heap; thousands of threads = gigabytes wasted on stacks alone. |
| 5 | **Operating System Scheduler** | The OS component deciding which thread runs on which core and when; parks blocked threads without wasting CPU on them. |
| 6 | **Context Switching** | The overhead of the CPU saving one thread's state and loading another's — too many threads means too much time spent switching, not working. |
| 7 | **Synchronous Programming** | Code runs one statement at a time; each step must fully finish (including waiting) before the next starts. |
| 8 | **Asynchronous Programming** | Code starts an operation and moves on immediately; the result arrives later via callback/event — no waiting in place. |
| 9 | **Blocking I/O** | The thread freezes until an I/O operation (DB call, file read, network call) completes. |
| 10 | **Non-Blocking I/O** | The thread doesn't wait; it's notified when I/O data is ready, freeing it to do other work meanwhile. |
| 11 | **Various I/O Models** | Blocking → Polling → I/O Multiplexing (epoll/kqueue, what Netty/WebFlux use) → Async I/O (OS-driven completion). |
| 12 | **Why Reactive Programming?** | To avoid wasting threads on I/O waits — declare a pipeline once, let the engine drive it efficiently with few threads. |
| 13 | **When to use Reactive Programming** | Best for high-concurrency, I/O-heavy workloads; a poor fit for CPU-bound work or low-concurrency simple apps. |
| 14 | **Common misconceptions** | Reactive ≠ automatically faster/parallel; one accidental blocking call can still stall the whole app; it doesn't fix bad architecture. |

## How It All Fits Together

```
Hardware/OS limits (threads, memory, context switches)
        │
        ▼
Blocking I/O wastes threads while waiting
        │
        ▼
Non-blocking I/O releases threads instead of freezing them
        │
        ▼
Reactive programming = a structured way to write non-blocking,
                        asynchronous pipelines (Mono/Flux) that
                        scale to huge concurrency with few threads
```

Once this mental model clicks, everything else in this course — `Mono`, `Flux`,
schedulers, backpressure — is just the practical toolkit for applying this idea in
real code.
