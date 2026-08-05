# Asynchronous Programming

## In Simple Terms

**Asynchronous** code kicks off an operation and **does not wait** for it to finish.
Instead, it moves on immediately, and the result (or error) is delivered later —
usually via a callback, a `Future`, or (in reactive programming) a stream of events.

The key shift in thinking: "call me back when you're done" instead of "I'll wait
right here until you're done."

## Simple Example

```java
public class AsyncDemo {
    public static void main(String[] args) {
        System.out.println("Step 1: Requesting user...");

        fetchUserAsync(userId, user -> {
            // this callback runs LATER, whenever the data is ready
            System.out.println("Got user: " + user.getName());
        });

        System.out.println("Step 2: This runs immediately, without waiting!");
    }
}
```

Notice "Step 2" prints **before** the user is fetched — the main thread never stopped
to wait. It's free to do other work while the fetch happens in the background.

## Why It Matters for Reactive Programming

Asynchronous programming is the foundation reactive programming builds on. But plain
async callbacks get messy fast when you need to chain many steps ("callback hell").
Project Reactor's `Mono`/`Flux` give you a clean, composable way to write
asynchronous, non-blocking pipelines without nesting callbacks endlessly.
