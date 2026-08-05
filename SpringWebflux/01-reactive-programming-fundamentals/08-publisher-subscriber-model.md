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

## Why It Matters

Understanding that "everything is a Publisher" in WebFlux — requests, responses,
database calls, external API calls — is the key mental model shift from traditional
Spring MVC. Once you see the whole request pipeline as a chain of publishers and
subscribers, WebFlux's behavior (laziness, backpressure, non-blocking flow) becomes
much more intuitive.
