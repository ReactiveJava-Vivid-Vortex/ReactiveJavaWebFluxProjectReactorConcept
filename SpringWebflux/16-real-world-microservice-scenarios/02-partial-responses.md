# Partial Responses

## In Simple Terms

When you're gathering data from several downstream services, sometimes
one of them fails while the others are fine. Instead of failing the whole
response, a well-designed service can send back a partial response — the
data that is available, with a clear sign of what's missing.

## Simple Example

```java
public record Dashboard(
    Optional<UserProfile> profile,
    Optional<List<Order>> recentOrders,
    Optional<Recommendations> recommendations
) {}

public Mono<Dashboard> getDashboard(String userId) {
    Mono<Optional<UserProfile>> profileMono = userService.getProfile(userId)
        .map(Optional::of)
        .onErrorReturn(Optional.empty());

    Mono<Optional<List<Order>>> ordersMono = orderService.getRecentOrders(userId)
        .map(Optional::of)
        .onErrorReturn(Optional.empty());

    Mono<Optional<Recommendations>> recsMono = recommendationService.getFor(userId)
        .map(Optional::of)
        .onErrorReturn(Optional.empty());

    return Mono.zip(profileMono, ordersMono, recsMono)
        .map(tuple -> new Dashboard(tuple.getT1(), tuple.getT2(), tuple.getT3()));
}
```

If the recommendation service is down, the dashboard still shows up with
the profile and order data — just with `recommendations` empty — instead
of failing the whole request.

## Why It Matters

Partial responses give a much better experience than an all-or-nothing
approach: a dashboard missing one non-critical widget beats a totally
blank error page, especially when the failing piece isn't essential to the
page as a whole.
