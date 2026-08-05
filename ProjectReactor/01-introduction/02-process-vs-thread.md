# Process vs Thread

## In Simple Terms

A **process** is a running program with its own private memory space — like a separate
apartment. Your browser is a process, your IDE is a process. Processes don't share
memory with each other directly; they are isolated.

A **thread** is a unit of execution *inside* a process — like a person living in that
apartment. A single process (apartment) can have many threads (people) working at the
same time, and they all share the same memory (the apartment's furniture, fridge,
etc.).

Because threads share memory, they can communicate easily, but they can also
interfere with each other (race conditions) if not coordinated carefully.

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

Both `main` and `worker` run inside the **same JVM process** and share the same heap
memory — that's why a variable created in one thread can be seen by another (if
shared correctly). If instead you ran two separate Java programs (`java App1` and
`java App2`), those would be two different **processes**, each with its own memory,
unable to see each other's variables directly.

## Why It Matters for Reactive Programming

Reactive programming is fundamentally about using a **small number of threads**
efficiently, instead of creating a new thread per task (which is expensive). Understanding
that threads are the actual "workers" executing your reactive pipeline is the first
building block.
