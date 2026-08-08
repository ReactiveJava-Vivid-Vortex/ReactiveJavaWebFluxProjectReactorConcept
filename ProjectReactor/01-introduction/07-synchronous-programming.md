# Synchronous Programming

## In Simple Terms

**Synchronous** code runs one line at a time, in order, and each line has to fully
finish before the next one starts. If one line takes 5 seconds (say, a slow
database call), the whole thread is stuck there for those 5 seconds, doing
nothing else.

This is how most people first learn to write code — it's simple and predictable.

## Simple Example

```java
public class SyncDemo {
    public static void main(String[] args) {
        System.out.println("Step 1: Fetching user...");
        User user = fetchUserFromDatabase(); // stuck here for, say, 2 seconds
        System.out.println("Step 2: Got user: " + user.getName());

        System.out.println("Step 3: Fetching orders...");
        List<Order> orders = fetchOrders(user); // stuck again
        System.out.println("Step 4: Got " + orders.size() + " orders");
    }
}
```

Each step has to fully wait for the one before it. The total time is just the sum
of every step — nothing happens in parallel.

## Why It Matters for Reactive Programming

Synchronous code is easy to follow, but it wastes the thread every time it waits.
Reactive programming pushes back on this: instead of the thread just sitting there
during step 2, it should be free to go do something else, and get pinged the
moment the user data is actually ready.
