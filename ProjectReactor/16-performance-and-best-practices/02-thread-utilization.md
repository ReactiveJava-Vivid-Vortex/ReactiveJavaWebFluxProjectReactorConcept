# Thread Utilization

## In Simple Terms

"Thread utilization" is just how well you're using the threads you have —
ideally, every thread should be doing real work almost all the time,
instead of just sitting around waiting for something slow to finish.
Reactive programming's biggest performance win comes from making thread
usage dramatically more efficient compared to the old one-thread-per-request
approach.

## Simple Example

Blocking model — poor use of threads under load:

```
1000 concurrent requests, each waiting 200ms on a DB call
-> needs up to 1000 threads (mostly idle, just waiting)
-> huge memory overhead, most threads doing nothing useful at any instant
```

Reactive model — much better use of threads:

```
1000 concurrent requests, using non-blocking I/O
-> handled by ~8-16 event-loop threads
-> those threads are constantly busy servicing whichever request has data ready
-> no thread sits idle waiting for a specific request's I/O to finish
```

## Why It Matters

Good thread utilization is exactly why a reactive server can handle way
more concurrent connections with far fewer threads (and therefore way less
memory) than an equivalent blocking server — the same hardware ends up
serving a lot more traffic, as long as blocking calls are properly isolated
(see [[non-blocking-execution]]).
