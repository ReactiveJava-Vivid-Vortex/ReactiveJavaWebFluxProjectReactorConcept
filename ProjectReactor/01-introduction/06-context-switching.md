# Context Switching

## In Simple Terms

A **context switch** is what happens when the CPU stops running one thread and
starts running another. To do that safely, it has to save exactly where the first
thread was (like bookmarking a page), then load where the next thread left off.
This bookkeeping takes real time — and while it's happening, **no actual work gets
done.**

The more threads competing for the same handful of CPU cores, the more switching
happens, and the more time gets wasted on bookkeeping instead of real work.

## Simple Example

Imagine you're reading one novel, and someone keeps interrupting you to read a
different book instead. Every time, you have to:

1. Remember your page number in book A.
2. Put a bookmark in.
3. Pick up book B and find your place.

Do this every few seconds, switching between many books, and eventually you spend
more time finding your place than actually reading. That's exactly what context
switching costs a CPU.

```
Too many threads  ->  too much switching  ->  less real work gets done
```

## Why It Matters for Reactive Programming

Reactive systems deliberately keep the number of active threads **small** —
usually close to the number of CPU cores — which keeps context switching to a
minimum. Instead of spinning up a fresh thread for every task, they line up the
work and process it efficiently with a handful of long-lived threads.
