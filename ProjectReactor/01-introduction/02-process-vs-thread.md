# Process vs Thread

## In Simple Terms

A **process** is a running program with its own private slice of memory — think of
it like an apartment. Your browser is one apartment, your IDE is another. They
can't just walk into each other's kitchen and grab a variable.

A **thread** is a worker living inside that apartment. One apartment (process) can
have several workers (threads) doing things at the same time, and they all share
the same furniture and fridge (memory).

Because threads share memory, they can pass information to each other easily. But
that also means two threads can bump into each other and cause bugs if they're not
careful — that's called a race condition.

## Simple Example

```java
public class ProcessVsThreadDemo {
    public static void main(String[] args) {
        // "main" is already a thread running inside the JVM process
        System.out.println("Running on thread: " + Thread.currentThread().getName());

        Thread worker = new Thread(() -> {
            System.out.println("Running on thread: " + Thread.currentThread().getName());
        });
        worker.start();
    }
}
```

Both `main` and `worker` live in the same apartment (JVM process) and share the
same fridge (heap memory) — that's why one thread can see a variable another
thread created. If you instead ran two separate `java` commands, those would be
two completely separate apartments, unable to see each other's stuff at all.

## Why It Matters for Reactive Programming

Reactive programming is really about getting a **small number of workers** to do
a lot of work efficiently, instead of hiring a brand-new worker for every single
task (which costs time and memory). Threads are the actual workers running your
reactive code — everything else in this course builds on that idea.
