# Non-Blocking I/O

## In Simple Terms

**Non-blocking I/O** means: when your thread asks for data, it does **not** wait. It
either gets told "no data yet, come back later" immediately, or it registers interest
in that I/O source and moves on to do other work. When the data finally becomes
available, the thread (or a different one) is notified and processes it.

This lets a **single thread** effectively juggle thousands of in-flight I/O
operations, because it's never frozen waiting on any one of them.

## Simple Example

```java
// Conceptual illustration (real NIO code is more verbose)
Channel channel = openNonBlockingChannel("example.com", 80);

channel.onDataAvailable(data -> {
    // This runs LATER, only when data has actually arrived
    System.out.println("Got data: " + data);
});

System.out.println("Registered interest, moving on immediately!");
// This thread is now free to service other channels/requests
```

Under the hood, the OS gives you a mechanism (like `epoll` on Linux, or `kqueue` on
macOS) that lets one thread ask: "of these 10,000 sockets I'm watching, which ones
have new data right now?" — instead of checking (or blocking on) each one individually.

## Why It Matters for Reactive Programming

Non-blocking I/O is the *technical mechanism* that makes reactive programming
possible at scale. Project Reactor and Spring WebFlux are built on top of non-blocking
I/O (via Netty), which is why a WebFlux app can serve tens of thousands of concurrent
connections using only a handful of threads.
