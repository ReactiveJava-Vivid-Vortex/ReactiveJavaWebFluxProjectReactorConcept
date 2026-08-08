# Q1. What Are Hot and Cold Publishers in Project Reactor?

## Simple Explanation (Think of YouTube)

Imagine there is a YouTube video.

### Cold Publisher = YouTube Recorded Video 📹

Every person watches from the beginning.

```
User A -----> Starts from 0:00
User B -----> Starts from 0:00
User C -----> Starts from 0:00
```

Everyone gets **their own copy** of the video.

Nobody affects anyone else.

---

### Hot Publisher = Live Cricket Match 🏏

The match is already happening.

```
Current Time = 45th over

User A joins now -> sees 45th over
User B joins later -> sees 48th over
User C joins later -> sees last over
```

Nobody can rewind.

Everyone only receives events **after they subscribe.**

---

That is the biggest difference.

| Cold Publisher                 | Hot Publisher                    |
| ------------------------------ | --------------------------------- |
| Starts when subscriber arrives | Already producing data           |
| Every subscriber gets all data | Subscriber only gets future data |
| Independent stream             | Shared stream                    |
| Like Netflix                   | Like Live TV                     |

---

# Q2. Why Are They Called "Publisher"?

Remember the Reactive Streams model.

```
Publisher
     ↓
Subscriber
```

Publisher produces data.

Subscriber consumes data.

Whether it is Hot or Cold simply defines **how data is produced.**

---

# Q3. What Is a Cold Publisher?

A Cold Publisher **does nothing until someone subscribes.**

Think of it as:

> "I'll start working only when somebody asks."

Example:

```java
Flux<Integer> flux =
    Flux.just(1,2,3,4);
```

Nothing has happened yet.

```
Flux created

No subscriber

No values produced
```

Only when:

```java
flux.subscribe();
```

does Reactor begin emitting.

---

Suppose two subscribers come.

```java
flux.subscribe(System.out::println);

flux.subscribe(System.out::println);
```

Output

```
Subscriber 1

1
2
3
4

Subscriber 2

1
2
3
4
```

Each subscriber receives the entire sequence.

---

Visualization

```
             Flux.just(1,2,3)

Subscriber A

1
2
3

Subscriber B

1
2
3
```

Two completely independent streams.

---

# Q4. Why Is It Called "Cold"?

Because nothing is happening.

Imagine an engine.

```
Car parked.

Engine OFF.

Nobody driving.
```

As soon as you sit inside,

```
Engine starts.
```

Exactly how Cold Publisher behaves.

---

# Q5. Examples of Cold Publishers

Most Reactor publishers are cold.

```
Flux.just()

Flux.range()

Flux.fromIterable()

Mono.just()

Mono.fromCallable()

Mono.defer()
```

Even database calls are typically cold.

Example

```java
Mono<User> user =
    repository.findById(10);
```

Query isn't executed until

```java
user.subscribe();
```

---

# Q6. What Is a Hot Publisher?

A Hot Publisher starts producing data **even if nobody is listening.**

Think of

* Live stock market
* Live GPS
* Live cricket
* Temperature sensor
* Mouse movement
* Kafka topic
* RabbitMQ queue
* WebSocket messages

They don't wait for you.

---

Example timeline

```
Time

0 1 2 3 4 5 6

Publisher

A B C D E F G

Subscriber joins here

          ↑

Receives

D E F G
```

A, B and C are gone forever.

---

# Q7. Why Is It Called "Hot"?

Imagine a running tap.

```
Water flowing...

Nobody collecting.
```

Later someone brings a bucket.

They only collect

```
Current water onward.
```

They cannot collect earlier water.

---

# Q8. Simple Java Example

Cold Publisher

```java
Flux<Integer> flux =
    Flux.range(1,5);

flux.subscribe(System.out::println);

Thread.sleep(3000);

flux.subscribe(System.out::println);
```

Output

```
Subscriber 1

1
2
3
4
5

Subscriber 2

1
2
3
4
5
```

Both receive everything.

---

Hot Publisher

```java
Sinks.Many<Integer> sink =
    Sinks.many().multicast().directBestEffort();

Flux<Integer> hotFlux =
    sink.asFlux();
```

Producer

```java
sink.tryEmitNext(1);

sink.tryEmitNext(2);
```

Now subscriber joins.

```java
hotFlux.subscribe(System.out::println);

sink.tryEmitNext(3);

sink.tryEmitNext(4);
```

Subscriber output

```
3
4
```

It never sees

```
1
2
```

because they were emitted before subscription.

---

# Q9. Which Publisher Is the Default in Reactor?

Almost everything in Reactor is **Cold**.

For example

```java
Flux.just()

Flux.range()

Flux.defer()

Mono.just()

Mono.fromCallable()

repository.findAll()
```

are all Cold Publishers.

Hot publishers are created explicitly using APIs like:

* `Sinks.Many`
* `publish().autoConnect()`
* `share()`
* External event sources (WebSocket, Kafka consumers, sensors)

---

# Q10. Real-Life Examples

## Cold

```
Reading a file

HTTP request

Database query

Generating PDF

Calling REST API
```

Each user performs their own work.

---

## Hot

```
Chat messages

Mouse events

Stock prices

Weather sensor

Kafka events

WebSocket

Application logs
```

Data exists independently of subscribers.

---

# Q11. Resource Usage

Cold Publisher

```
Subscriber A

Database Query

Subscriber B

Database Query

Subscriber C

Database Query
```

Three subscribers

↓

Three database queries

---

Hot Publisher

```
Database Query

↓

Shared

↓

Subscriber A

Subscriber B

Subscriber C
```

Only one upstream execution can serve multiple subscribers (depending on how the hot publisher is created).

---

# Q12. Which One Replays Data?

Cold Publisher

Always replays.

```
Subscriber A

1
2
3

Subscriber B

1
2
3
```

---

Hot Publisher

Normally

```
Subscriber A

1
2
3

Subscriber B joins

Receives only

3
```

Earlier values are lost unless replay behavior is explicitly configured.

---

# Q13. Can a Cold Publisher Become Hot?

Yes.

Example

```java
Flux<Integer> cold =
    Flux.range(1,5);

Flux<Integer> hot =
    cold.share();
```

`share()` allows multiple subscribers to share a single upstream subscription while it is active.

Similarly,

```java
publish()
autoConnect()
refCount()
```

can convert a cold source into a shared (hot-like) publisher.

---

# Q14. What About `cache()`?

`cache()` is slightly different.

```
Subscriber A

1
2
3

Values cached

Subscriber B

Immediately receives

1
2
3
```

Unlike a normal hot publisher, cached values are replayed to later subscribers.

---

# Q15. Hot Publisher Types in Reactor

There are several behaviors depending on the operator or sink:

| Type      | Behavior                                                                               |
| --------- | ---------------------------------------------------------------------------------------- |
| Multicast | Only current subscribers receive new events. Late subscribers miss earlier ones.       |
| Replay    | Remembers previous items and replays them to late subscribers.                         |
| Cache     | Stores emitted items and serves them to future subscribers after the source completes. |
| Unicast   | Only one subscriber is allowed.                                                        |
| Latest    | Keeps only the most recent value for new subscribers (depending on the sink/operator). |

---

# Q16. When Should You Use Each?

Use a **Cold Publisher** when:

* Every subscriber should receive the complete data.
* Each subscription should execute independently.
* You're wrapping database calls, REST APIs, or file reads.

Use a **Hot Publisher** when:

* Events are happening continuously.
* Multiple subscribers should observe the same live stream.
* You're working with WebSockets, Kafka, sensors, UI events, or live notifications.

---

# Q17. Common Interview Questions

### Does a Cold Publisher execute before `subscribe()`?

**No.** It is lazy and starts only when subscribed.

---

### Can multiple subscribers receive the same values from a Cold Publisher?

**Yes.** Each subscriber gets a fresh execution and the full sequence.

---

### Can a Hot Publisher lose events?

**Yes.** If values are emitted before a subscriber joins, those values are typically missed unless replay or caching is configured.

---

### Is `Flux.just()` hot or cold?

**Cold.** Each subscription receives the full sequence.

---

### Is `Mono.just()` hot or cold?

**Cold.** Although it already holds a value internally, it emits that value separately for each subscription.

---

### Are Kafka messages hot?

Yes. Kafka is generally treated as a hot event source because messages are produced independently of your subscribers. (How much history a consumer can read depends on Kafka offsets, which is a Kafka feature rather than Reactor itself.)

---

# Q18. Summary

| Feature              | Cold Publisher                                                 | Hot Publisher                                                    |
| --------------------- | ---------------------------------------------------------------- | -------------------------------------------------------------------- |
| Starts producing     | On `subscribe()`                                               | Independently of subscribers                                     |
| Default in Reactor   | ✅ Yes                                                          | ❌ No (must be created or adapted)                                |
| Each subscriber gets | A fresh execution                                              | A shared live stream                                             |
| Late subscriber      | Receives everything from the start                             | Misses past events by default                                    |
| Re-executes upstream | Yes, once per subscription                                     | Typically no; subscribers share the same upstream                |
| Typical sources      | `Flux.just`, `Flux.range`, `Mono.just`, DB queries, REST calls | `Sinks.Many`, `share()`, `publish()`, WebSockets, Kafka, sensors |
| Best for             | Request/response, finite data                                  | Live events and broadcasts                                       |

### One sentence to remember

* **Cold Publisher:** *"Start producing when someone subscribes; every subscriber gets their own complete stream."*
* **Hot Publisher:** *"Produce continuously; subscribers only see events from the moment they join unless replay is explicitly enabled."*
