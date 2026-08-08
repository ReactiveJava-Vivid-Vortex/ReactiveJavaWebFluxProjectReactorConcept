# Non-Blocking I/O

## In Simple Terms

**Non-blocking I/O** means: when your thread asks for data, it does **not** just
sit there. It either gets told "not ready yet, I'll let you know," or it moves on
right away and gets notified later when the data finally shows up.

This means **one single thread** can juggle thousands of ongoing requests at
once, because it's never frozen waiting on any one of them.

## Simple Example

```java
// Conceptual illustration (real NIO code looks more verbose)
Channel channel = openNonBlockingChannel("example.com", 80);

channel.onDataAvailable(data -> {
    // This runs LATER, only once data has actually arrived
    System.out.println("Got data: " + data);
});

System.out.println("Registered interest, moving on immediately!");
// This thread is free to go help with other requests now
```

Under the hood, the operating system gives you a trick (called `epoll` on Linux,
`kqueue` on macOS) where one thread can ask: "of these 10,000 open connections,
which ones actually have new data right now?" — instead of checking (or freezing
on) each one individually.

## Why It Matters for Reactive Programming

Non-blocking I/O is the actual engine that makes reactive programming work at
scale. Project Reactor and Spring WebFlux run on top of it (via Netty), which is
why a WebFlux app can handle tens of thousands of connections using just a
handful of threads.
