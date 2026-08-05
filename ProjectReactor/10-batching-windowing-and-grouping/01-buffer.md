# buffer()

## In Simple Terms

`.buffer(n)` collects items into `List`s of a fixed size, emitting a new `List` every
time `n` items have accumulated. Instead of processing items one at a time, you get
them grouped into batches — useful for reducing the number of downstream operations
(e.g., doing one bulk database insert instead of many single-row inserts).

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

Note the last batch can be smaller if the total count isn't evenly divisible.

A practical example — batching records for bulk insert:

```java
recordFlux
    .buffer(100) // gather up to 100 records at a time
    .flatMap(batch -> database.bulkInsert(batch))
    .subscribe();
```

## Why It Matters

`.buffer()` is a key tool for **efficiency**: instead of making one network/database
call per item (slow, chatty), you batch items together and make far fewer, larger
calls — a huge performance win for high-volume data processing pipelines.
