# Publisher/Subscriber Model

## In Simple Terms

The Publisher/Subscriber model is the foundational pattern underlying all of Spring
WebFlux: a **Publisher** (like `Mono`/`Flux`) produces data over time, and a
**Subscriber** consumes it, reacting to new values, errors, or completion. WebFlux
controllers, `WebClient`, and R2DBC repositories all speak this same publisher-based
language throughout the entire request lifecycle.

## Simple Example

```java
@RestController
public class UserController {

    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable String id) { // Mono = a Publisher of 0-1 User
        return userRepository.findById(id); // WebFlux subscribes to this internally
    }

    @GetMapping("/users")
    public Flux<User> getAllUsers() { // Flux = a Publisher of 0-N Users
        return userRepository.findAll();
    }
}
```

You never call `.subscribe()` yourself in a controller — Spring WebFlux's internal
machinery subscribes to the `Mono`/`Flux` you return, at the right time, and streams
the resulting data back to the HTTP client as it becomes available.

## The Three Signals Every Publisher Speaks

Whatever the `Publisher` is — a `Mono<User>` from a repository, a `Flux<Order>`
streamed over SSE, a `WebClient` call to another service — it can only ever
communicate using **three signal types**:

- **`onNext(item)`** — "here's a value" (0+ times).
- **`onComplete()`** — "finished successfully" (terminal, at most once).
- **`onError(throwable)`** — "it failed" (terminal, at most once, mutually
  exclusive with `onComplete()`).

```java
userRepository.findById(id).subscribe(
    user -> System.out.println("onNext: " + user),   // 0 or 1 time (it's a Mono)
    err  -> System.out.println("onError: " + err),   // if the lookup fails
    ()   -> System.out.println("onComplete!")         // always fires last, on success
);
```

Spring WebFlux itself is built entirely on top of this signal grammar: a
`404 Not Found` for an empty `Mono` and a `500` for an unhandled `onError` are
just the framework's default reactions to which terminal signal your controller's
`Mono`/`Flux` ended with. See [[the-three-signal-types]] in the Project Reactor
notes for the full breakdown.

## Why It Matters

Understanding that "everything is a Publisher" in WebFlux — requests, responses,
database calls, external API calls — is the key mental model shift from traditional
Spring MVC. Once you see the whole request pipeline as a chain of publishers and
subscribers, WebFlux's behavior (laziness, backpressure, non-blocking flow) becomes
much more intuitive.
