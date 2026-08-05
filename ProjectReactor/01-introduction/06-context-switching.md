# Context Switching

## In Simple Terms

A **context switch** happens when the CPU stops running one thread and starts running
another. To do this safely, the OS must save the current thread's state (register
values, program counter, stack pointer) and load the next thread's saved state. This
"save and restore" work takes real time — typically microseconds — during which
**no actual application code runs**.

The more threads you have competing for a limited number of CPU cores, the more
context switching happens, and the more time is "wasted" on bookkeeping instead of
real work.

## Simple Example

Imagine you're reading a novel (Thread-A) and someone interrupts you to read a
different book (Thread-B). Before you can help them, you have to:

1. Remember your page number in novel A (save state).
2. Put a bookmark in novel A.
3. Pick up book B and find where you left off (load state).

Do this constantly, switching every few seconds between many books, and you'll notice
you spend more time finding your place than actually reading. That overhead is
context switching.

```
Too many threads  ->  too much switching  ->  less real work gets done
```

## Why It Matters for Reactive Programming

Reactive systems intentionally keep the **number of active threads small** (often
close to the number of CPU cores), which minimizes context switching. Instead of
creating a new thread for every task, they queue up work and process it efficiently
on a handful of long-lived worker threads.
