# Use Cases (Batching, Windowing & Grouping)

## In Simple Terms

These operators aren't just theory — they map straight onto very common
data-engineering patterns, especially in streaming and Kafka-style systems.

## 1. Kafka Message Batching

Instead of handling one Kafka message at a time, group them together for
efficiency:

```java
kafkaMessageFlux
    .bufferTimeout(500, Duration.ofSeconds(1)) // batch up to 500 msgs, or every 1s
    .flatMap(batch -> processBatchAndAcknowledge(batch))
    .subscribe();
```

## 2. Bulk Database Inserts

```java
recordFlux
    .buffer(1000)
    .flatMap(batch -> r2dbcTemplate.insertBatch(batch))
    .subscribe();
```

## 3. Revenue Calculation (Windowed Aggregation)

```java
transactionFlux
    .window(Duration.ofMinutes(1)) // 1-minute rolling windows
    .flatMap(window -> window.map(Transaction::getAmount).reduce(0.0, Double::sum))
    .subscribe(total -> System.out.println("Revenue this minute: " + total));
```

## 4. Log Processing

```java
logEventFlux
    .groupBy(LogEvent::getSeverity)
    .flatMap(group -> group.buffer(100)
        .doOnNext(batch -> shipToMonitoringSystem(group.key(), batch)))
    .subscribe();
```

## 5. Stream Partitioning

```java
eventFlux
    .groupBy(event -> event.getUserId().hashCode() % partitionCount)
    .flatMap(partition -> processPartition(partition))
    .subscribe();
```

## Why It Matters

Batching, windowing, and grouping are the backbone of building fast,
high-volume streaming systems — cutting down the number of expensive
downstream operations (database writes, network calls) while still keeping
data processing close to real time.
