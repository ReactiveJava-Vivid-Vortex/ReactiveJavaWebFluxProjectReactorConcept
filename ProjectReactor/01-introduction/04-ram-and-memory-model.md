# RAM and Memory Model

## In Simple Terms

RAM is where your program's data lives while it runs — variables, objects,
everything. Every thread gets its **own small notepad** (called a stack) for its
local variables, but all threads in the same program share **one big shared
whiteboard** (called the heap) for anything created with `new`.

Here's the surprising part: just having a thread exist costs memory. The JVM
usually reserves around 512KB–1MB per thread for its notepad. That sounds small —
until you create **10,000 threads**, and suddenly you've burned several gigabytes
of RAM before doing any real work.

## Simple Example

```java
public class MemoryDemo {
    static int sharedCounter = 0; // lives on the shared whiteboard (heap)

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            int localVar = 42; // lives on THIS thread's own notepad (stack)
            sharedCounter++;   // scribbles on the shared whiteboard
        };

        Thread t1 = new Thread(task);
        Thread t2 = new Thread(task);
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(sharedCounter); // both threads touched this
    }
}
```

## Why It Matters for Reactive Programming

Old-school servers often give **one thread per request** and make that thread
just sit and wait for the whole request. Under heavy load — say 50,000 requests at
once — that's 50,000 notepads, an enormous and often impossible amount of RAM.
Reactive programming avoids all that by handling many requests with a small,
fixed group of threads, since no thread ever just sits there waiting.
