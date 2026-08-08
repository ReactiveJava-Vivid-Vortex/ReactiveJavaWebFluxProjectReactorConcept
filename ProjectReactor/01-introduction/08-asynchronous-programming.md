# Asynchronous Programming

## In Simple Terms

**Asynchronous** code starts a task and then **doesn't wait around** for it. It
just moves on, and whenever the result (or an error) is ready, it gets delivered
later — usually through a callback, a `Future`, or, in reactive code, a stream of
events.

The mental shift is: instead of "I'll wait right here until you're done," it
becomes "call me back whenever you're done."

## Simple Example

```java
public class AsyncDemo {
    public static void main(String[] args) {
        System.out.println("Step 1: Requesting user...");

        fetchUserAsync(userId, user -> {
            // this runs LATER, whenever the data actually shows up
            System.out.println("Got user: " + user.getName());
        });

        System.out.println("Step 2: This runs immediately, without waiting!");
    }
}
```

Notice "Step 2" prints **before** the user has even been fetched — the main
thread never stopped to wait. It's free to keep working while the fetch happens
in the background.

## Why It Matters for Reactive Programming

Asynchronous programming is the foundation everything else here is built on. But
plain callbacks get messy fast once you chain several steps together (people call
this "callback hell"). `Mono`/`Flux` in Project Reactor give you a clean way to
write async, non-blocking code without nesting callback after callback.
