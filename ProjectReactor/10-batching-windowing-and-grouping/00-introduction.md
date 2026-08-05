# Batching, Windowing & Grouping — Topic Overview

## What Is This Topic About? (In Simple Terms)

Sometimes it's wasteful to process items strictly one at a time — imagine writing
one row at a time to a database versus batching 1,000 rows into a single bulk
insert. This topic covers three related ways to reorganize a stream of individual
items into more efficient, meaningful chunks.

- **`buffer(n)`** collects items into `List`s of a fixed size — simplest, but you
  get a fully materialized `List`.
- **`window(n)`** is similar, but each chunk stays a `Flux` (a stream) instead of a
  `List` — useful when a chunk itself might be huge, or you want to apply further
  reactive operators to it.
- **`groupBy(keyFn)`** partitions a stream into multiple sub-streams by key — like
  SQL's `GROUP BY` — one `GroupedFlux` per distinct key value.

```java
recordFlux
    .buffer(1000)                          // gather up to 1000 records
    .flatMap(batch -> database.bulkInsert(batch)) // one bulk call instead of 1000 single ones
    .subscribe();
```

Both `buffer()` and `window()` have `*Timeout` variants (`bufferTimeout()`,
`windowTimeout()`) that also close a chunk after a time limit — essential when
items trickle in slowly and you don't want to wait forever for a chunk to fill up.

**Important gotcha with `groupBy()`:** each `GroupedFlux` must actually be
subscribed to (usually via `.flatMap()`) — an ignored group can silently buffer
items in memory forever.

## Quick Revision Cheat Sheet

| # | Concept | One-Line Summary |
|---|---|---|
| 1 | **buffer()** | Collects items into fixed-size `List`s — simple batching for bulk operations. |
| 2 | **bufferTimeout()** | Like `buffer()`, but also closes a batch after a time limit — avoids waiting forever on slow streams. |
| 3 | **window()** | Like `buffer()`, but each chunk stays a `Flux` (streamable), not a materialized `List`. |
| 4 | **windowTimeout()** | Time-bounded version of `window()` — same size-or-time cutoff logic as `bufferTimeout()`. |
| 5 | **groupBy()** | Partitions a stream into multiple `GroupedFlux` sub-streams by key, like SQL `GROUP BY`. |
| 6 | **Use Cases** | Kafka batching, bulk DB inserts, windowed revenue calc, log processing, stream partitioning. |

## How It All Fits Together

```
Do you need FIXED-SIZE chunks for bulk operations?
   │
   ├── Chunk as a List?          ──▶ buffer(n)          [+ bufferTimeout() if slow/irregular arrival]
   ├── Chunk as a stream (Flux)? ──▶ window(n)          [+ windowTimeout() if slow/irregular arrival]
   │
   └── Need to SPLIT by a key instead (not by count/time)?
                                  ──▶ groupBy(keyFn)  →  must flatMap() each GroupedFlux!
```

The unifying theme: don't process a firehose of individual items one at a time when
your downstream system (a database, an API, a partition) is much happier receiving
organized chunks — batching and windowing turn "many small, chatty calls" into
"fewer, larger, efficient ones."
