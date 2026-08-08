# zipWith()

## In Simple Terms

`.zipWith()` does the same pairing job as `Flux.zip()`, just written as a
fluent chain (`a.zipWith(b)`) instead of a static call — one item from each
side, paired up, one to one.

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

With a custom combining function so you don't have to deal with raw tuples:

```java
userMono.zipWith(ordersMono, (user, orders) ->
    new UserProfile(user, orders)
).subscribe(profile -> System.out.println(profile));
```

## Why It Matters

`.zipWith()` shows up constantly in service layers — fetching two related
pieces of data at the same time (instead of one after the other) and
combining them the moment both are ready. That roughly cuts your total wait
time in half compared to fetching them one at a time.
