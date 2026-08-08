# Q1. What Is Reactive Programming, and Why Does It Even Exist?

## Simple Explanation (Think of a Restaurant)

Imagine a restaurant with **one waiter** and many tables.

### Blocking Waiter = Traditional (Synchronous) Server 🧍

```
Table A orders food
Waiter walks order to kitchen
Waiter STANDS THERE waiting for food to be ready
Waiter serves Table A
Waiter FINALLY goes to Table B
```

While the waiter is standing at the kitchen window, tables B, C, D are sitting
there, ignored — not because the waiter is lazy, but because that's literally how
this waiter is built to work: one thing at a time, wait fully, then move on.

### Non-Blocking Waiter = Reactive Server ⚡

```
Table A orders food -> Waiter drops the order at the kitchen, walks away immediately
Table B orders food -> Waiter drops the order at the kitchen, walks away immediately
Table C orders food -> Waiter drops the order at the kitchen, walks away immediately
Kitchen bell rings (Table A's food ready) -> Waiter delivers it, THEN moves on
```

**Same one waiter.** Far more tables served. Nobody was ever "frozen" waiting.

That one idea — *don't wait in place, get notified instead* — is the entire seed
of reactive programming.

---

## Q2. Why Does "Waiting" Even Cost Anything? (The Hardware Story)

A CPU core runs exactly **one instruction stream at a time.** Your OS creates the
illusion of many programs running "at once" by rapidly switching a core between
threads — a few milliseconds each.

```
Time slice 1: CPU runs Thread-A
Time slice 2: CPU runs Thread-B
Time slice 3: CPU runs Thread-C
Time slice 4: CPU runs Thread-A again
...
```

Every thread — even one doing nothing but "waiting" — still needs:

- Its own memory stack (~512KB–1MB by default in the JVM)
- To be tracked and scheduled by the OS
- A "context switch" cost every time the CPU hands control to/from it

```
10,000 waiting threads  ->  ~10 GB just for stacks, doing ZERO useful work
```

**This is the actual, physical cost that reactive programming is designed to
avoid.**

---

## Q3. Process vs Thread — What's the Difference?

| Process | Thread |
|---|---|
| A running program with its **own private memory** | A worker **inside** a process |
| Isolated — can't see another process's memory directly | Shares memory with other threads in the same process |
| Your browser, your IDE = separate processes | `main` thread + worker threads inside one JVM |

```java
Thread worker = new Thread(() -> System.out.println("I'm a thread, not a process"));
worker.start();
// Still runs inside the SAME JVM process as "main"
```

Reactive programming is fundamentally about using a **small number of threads**
efficiently — not spinning up a new thread per task.

---

## Q4. Synchronous vs Asynchronous — What's Actually Different?

**Synchronous:** one statement at a time; each must fully finish (including any
waiting) before the next starts.

```java
System.out.println("Step 1: fetching user...");
User user = fetchUserFromDatabase(); // BLOCKS here for 2 seconds
System.out.println("Step 2: got " + user.getName());
```

**Asynchronous:** kick off the operation, move on immediately, get the result
*later* via a callback/event.

```java
System.out.println("Step 1: requesting user...");
fetchUserAsync(id, user -> System.out.println("Got: " + user.getName())); // callback, later
System.out.println("Step 2: runs IMMEDIATELY, without waiting!");
```

---

## Q5. Blocking vs Non-Blocking I/O — What's Actually Different?

| Blocking I/O | Non-Blocking I/O |
|---|---|
| Thread **freezes** until data arrives | Thread is told "not ready yet" or gets notified later |
| One thread can only handle one slow operation at a time | One thread can juggle **thousands** of in-flight operations |
| `in.read()` on a plain socket | `epoll`/`kqueue`/NIO selectors (what Netty/WebFlux use) |

```java
// Blocking — this thread does NOTHING else for however long this takes
int data = socket.getInputStream().read();
```

```
Non-blocking (conceptual):
  Register interest in 10,000 sockets
  OS says: "these 3 have data ready right now"
  Handle just those 3, then ask again
  -> ONE thread can service thousands of connections
```

---

## Q6. The Various I/O Models, Side by Side

```
1. Blocking I/O        Ask -> WAIT (frozen) -> Get Data
2. Polling              Ask -> "not ready" -> Ask again -> ... -> Get Data
3. I/O Multiplexing     Watch 1000 sockets -> OS says "these 3 are ready" -> Read those 3
4. Async I/O (AIO)      Ask -> OS does it fully in background -> "Here's your data" (callback)
```

Spring WebFlux runs on **Netty**, which uses model #3 (I/O multiplexing) — this is
exactly why a handful of threads can serve tens of thousands of connections.

---

## Q7. So... Why Reactive Programming? (Putting It All Together)

```
Blocking server:
  10,000 concurrent slow requests -> needs up to 10,000 threads
  -> gigabytes wasted on idle stacks -> server runs out of resources

Reactive server:
  10,000 concurrent slow requests -> handled by ~8-16 event-loop threads
  -> threads NEVER sit idle waiting -> same hardware, vastly more capacity
```

```java
// Blocking — thread frozen until the DB responds
@GetMapping("/user/{id}")
public User getUser(@PathVariable String id) { return userRepository.findById(id); }

// Reactive — thread released immediately, resumes when data is ready
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable String id) { return userRepository.findById(id); }
```

---

## Q8. When SHOULD You Use Reactive Programming?

| Use Case | Good Fit? |
|---|---|
| Public API gateway, 50k req/sec, I/O heavy | ✅ Yes |
| Streaming live prices/events to browsers | ✅ Yes |
| Aggregating calls to 5 downstream microservices | ✅ Yes |
| Batch job resizing 10,000 images (CPU-bound) | ❌ No — reactive adds no benefit here |
| Small internal admin tool, 10 users | ❌ No — added complexity isn't worth it |

**Rule of thumb:** reactive wins when you have **high concurrency + I/O waiting**.
It does nothing for pure CPU-bound work, and it adds real learning-curve cost —
don't reach for it by default.

---

## Q9. Common Misconceptions (Interview-Style Q&A)

### Does reactive programming make a single operation faster?

**No.** A single reactive DB call isn't quicker than a blocking one — the network/DB
latency is identical either way. Reactive improves **throughput under high
concurrency**, not per-operation speed.

### Does reactive code automatically run in parallel / on multiple threads?

**No.** By default, a reactive pipeline runs on whatever thread called
`subscribe()`, sequentially. You must explicitly use `subscribeOn()`/`publishOn()`
to move work to another thread.

### Is it fine to sneak in one blocking call inside a reactive pipeline "since it's just one line"?

**No.** A single blocking call can freeze one of your few precious event-loop
threads, stalling many *unrelated* requests sharing that thread — one of the most
dangerous, easy-to-miss mistakes in reactive code.

### Does reactive programming fix bad architecture?

**No.** If your database is slow, reactive code still waits on it — it just
doesn't waste a thread while doing so.

---

## Q10. Summary

| Concept | Key Takeaway |
|---|---|
| Process vs Thread | Process = isolated memory; Thread = worker sharing memory within a process |
| CPU/OS Scheduling | Threads compete for CPU time; waiting threads still cost memory + scheduling overhead |
| Context Switching | Too many threads = time wasted switching, not working |
| Blocking I/O | Thread freezes until data arrives — wasteful under high concurrency |
| Non-Blocking I/O | Thread is freed instantly, notified later — the mechanism reactive relies on |
| Reactive Programming | A structured way to write non-blocking pipelines that scale to huge concurrency with few threads |
| Best fit | High-concurrency, I/O-heavy workloads — NOT CPU-bound or low-concurrency apps |

### One sentence to remember

> **"Reactive programming exists so that a thread never sits idle waiting — it's
> released instantly and put back to work the moment it would otherwise freeze."**
