# Why Reactive Programming?

## In Simple Terms

Reactive programming exists to solve one specific problem: **traditional blocking,
thread-per-request applications don't scale well when there is a lot of waiting on
I/O** (databases, external APIs, file systems). Every waiting thread costs memory and
adds scheduling overhead, even though it's doing zero useful work.

Reactive programming flips the model: instead of a thread pulling data and waiting,
your code **declares a pipeline** of what should happen when data becomes available,
errors occur, or the stream completes — and the underlying engine (Project Reactor)
drives that pipeline using a small, efficient set of threads, only doing work when
there's actually something to do.

## Simple Example

Blocking approach (each request ties up a thread for the full duration):

```java
@GetMapping("/user/{id}")
public User getUser(@PathVariable String id) {
    return userRepository.findById(id); // thread blocks until the DB responds
}
```

Reactive approach (thread is released immediately; work resumes on data arrival):

```java
@GetMapping("/user/{id}")
public Mono<User> getUser(@PathVariable String id) {
    return userRepository.findById(id); // returns immediately; no thread is blocked
}
```

In the reactive version, while waiting for the database, the thread that started the
request is free to handle other incoming requests. When the DB responds, whichever
available thread is free continues the pipeline.

## Why It Matters

With reactive programming, a small server (say, 8 threads) can comfortably serve tens
of thousands of concurrent slow I/O-bound requests — something that would require
tens of thousands of threads (and gigabytes of extra RAM) in the blocking model.
