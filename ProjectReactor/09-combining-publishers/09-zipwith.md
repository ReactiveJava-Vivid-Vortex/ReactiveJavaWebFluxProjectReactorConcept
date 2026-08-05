# zipWith()

## In Simple Terms

`.zipWith(other)` is the instance-method version of `Flux.zip()` — it pairs the
current publisher's items with another publisher's items, one-to-one, written as a
fluent chain instead of a static method call.

## Simple Example

```java
Mono<String> userMono = userService.getUser(userId);
Mono<List<Order>> ordersMono = orderService.getOrders(userId);

userMono.zipWith(ordersMono)
    .subscribe(tuple -> {
        String user = tuple.getT1();
        List<Order> orders = tuple.getT2();
        System.out.println(user + " has " + orders.size() + " orders");
    });
```

With a custom combiner function to avoid dealing with raw tuples:

```java
userMono.zipWith(ordersMono, (user, orders) ->
    new UserProfile(user, orders)
).subscribe(profile -> System.out.println(profile));
```

## Why It Matters

`.zipWith()` is very commonly used in service layers to fetch two related pieces of
data **concurrently** (rather than sequentially, one after another) and combine them
once both are ready — cutting total latency roughly in half compared to fetching them
one at a time.
