# Why Reactive Programming?

## In Simple Terms

Reactive programming exists to fix one specific problem: **regular
one-thread-per-request apps don't scale well when there's a lot of waiting on
slow things** — databases, other services, files. Every thread that's just
sitting around waiting still costs memory and management overhead, even though
it's doing zero real work.

Reactive programming flips this around. Instead of a thread grabbing data and
waiting for it, your code describes a **pipeline**: what should happen when data
shows up, when something fails, or when it's all done. Project Reactor then runs
that pipeline using a small, efficient group of threads — only doing work when
there's actually something to do.

## Simple Example

The old way — each request holds its thread hostage the whole time:

```java
@GetMapping("/user/{id}")
public User getUser(@PathVariable String id) {
    return userRepository.findById(id); // thread waits here until the DB replies
}
```

The reactive way — the thread is freed up right away:

```java
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable String id) {
    return userRepository.findById(id); // returns instantly; no thread sits waiting
}
```

In the reactive version, while the database is working, that thread goes and
helps another request. Whenever the database answers, any free thread picks the
work back up.

## Why It Matters

With reactive programming, a small server — even just 8 threads — can comfortably
handle tens of thousands of slow, concurrent requests. Doing the same thing the
old way would need tens of thousands of threads and gigabytes of extra memory.
