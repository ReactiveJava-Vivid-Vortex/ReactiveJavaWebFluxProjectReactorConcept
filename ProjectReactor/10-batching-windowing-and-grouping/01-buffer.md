# buffer()

## In Simple Terms

`.buffer(n)` gathers items into little groups of a fixed size instead of
handing them to you one at a time — like a cashier who waits for 3
customers to line up before ringing them through together instead of
processing each one separately. Every time `n` items pile up, you get a
`List` with all of them.

## Simple Example

```java
Flux.range(1, 10)
    .buffer(3)
    .subscribe(batch -> System.out.println("Batch: " + batch));
```

Output:
```
Batch: [1, 2, 3]
Batch: [4, 5, 6]
Batch: [7, 8, 9]
Batch: [10]
```

Notice the last batch can be smaller if things don't divide evenly.

A practical example — grouping records together for a bulk insert:

```java
recordFlux
    .buffer(100) // gather up to 100 records at a time
    .flatMap(batch -> database.bulkInsert(batch))
    .subscribe();
```

## Why It Matters

`.buffer()` is a big deal for efficiency: instead of making one network or
database call per item (slow and chatty), you group items together and
make far fewer, bigger calls — a real performance win for anything handling
lots of data.
