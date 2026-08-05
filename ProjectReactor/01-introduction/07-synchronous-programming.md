# Synchronous Programming

## In Simple Terms

**Synchronous** code runs **one statement at a time, in order**, and each statement
must finish before the next one starts. If a statement takes 5 seconds (e.g., a slow
database call), the entire thread is stuck there for those 5 seconds — nothing else
on that thread can happen.

This is the default, most intuitive way most people first learn to code.

## Simple Example

```java
public class SyncDemo {
    public static void main(String[] args) {
        System.out.println("Step 1: Fetching user...");
        User user = fetchUserFromDatabase(); // blocks here for, say, 2 seconds
        System.out.println("Step 2: Got user: " + user.getName());

        System.out.println("Step 3: Fetching orders...");
        List<Order> orders = fetchOrders(user); // blocks again
        System.out.println("Step 4: Got " + orders.size() + " orders");
    }
}
```

Each step waits for the previous one. The total time is the **sum** of every step's
duration — nothing overlaps.

## Why It Matters for Reactive Programming

Synchronous code is easy to read and reason about, but it wastes the calling thread
during every wait. Reactive programming challenges this model: instead of the thread
"parking" during step 2, it should be free to do other work, and get notified when
the user data is ready.
