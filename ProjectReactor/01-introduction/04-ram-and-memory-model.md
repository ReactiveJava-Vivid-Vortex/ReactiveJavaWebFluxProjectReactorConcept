# RAM and Memory Model

## In Simple Terms

RAM (Random Access Memory) is where your running program's data lives — variables,
objects, the call stack. Every thread in a process gets its **own stack** (for local
variables and method calls) but shares the **same heap** (for objects created with
`new`).

Each thread also needs its own memory just to *exist* — the stack typically reserves
around 512KB–1MB by default in the JVM. This might sound small, but if you create
**10,000 threads**, that's several gigabytes of RAM just for stacks, before any actual
business logic runs.

## Simple Example

```java
public class MemoryDemo {
    static int sharedCounter = 0; // lives on the heap, shared by all threads

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            int localVar = 42; // lives on THIS thread's own stack
            sharedCounter++;   // modifies shared heap memory
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(sharedCounter); // shared state, both threads touched it
    }
}
```

## Why It Matters for Reactive Programming

Traditional blocking servers often use a "thread-per-request" model: one thread
waits, doing nothing, for each in-flight HTTP request. Under heavy load (say, 50,000
concurrent requests), that would need 50,000 threads — an enormous, often impossible,
amount of RAM. Reactive programming avoids this by handling many requests with a
**small, fixed pool of threads**, since no thread ever sits idle waiting.
