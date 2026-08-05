# Thread Utilization

## In Simple Terms

"Thread utilization" measures how effectively your available threads are being used
— ideally, every thread should be doing real work almost all the time, rather than
sitting idle waiting on I/O. Reactive programming's main performance win comes from
dramatically improving thread utilization compared to blocking, thread-per-request
models.

## Simple Example

Blocking model — poor utilization under high concurrency:

```
1000 concurrent requests, each waiting 200ms on a DB call
-> needs up to 1000 threads (mostly idle, just waiting)
-> huge memory overhead, most threads doing nothing useful at any instant
```

Reactive model — high utilization:

```
1000 concurrent requests, using non-blocking I/O
-> handled by ~8-16 event-loop threads
-> those threads are constantly busy servicing whichever request has data ready
-> no thread sits idle waiting for a specific request's I/O to finish
```

## Why It Matters

High thread utilization is precisely why a reactive server can handle vastly more
concurrent connections with far fewer threads (and therefore far less memory) than
an equivalent blocking server — the same hardware serves dramatically more traffic,
provided blocking calls are correctly isolated (see [[non-blocking-execution]]).
