# Filters (WebClient)

## In Simple Terms

`WebClient` supports filters (through `ExchangeFilterFunction`) — a way to
intercept and tweak every outgoing request and incoming response, similar
in spirit to `WebFilter` for incoming server requests. Common uses include
adding auth headers, logging, or retry logic uniformly across every call
made through a given `WebClient`.

## Simple Example

```java
ExchangeFilterFunction addAuthHeader = ExchangeFilterFunction.ofRequestProcessor(request ->
    Mono.just(ClientRequest.from(request)
        .header("Authorization", "Bearer " + getCurrentToken())
        .build())
);

ExchangeFilterFunction logResponse = ExchangeFilterFunction.ofResponseProcessor(response -> {
    System.out.println("Response status: " + response.statusCode());
    return Mono.just(response);
});

WebClient webClient = WebClient.builder()
    .baseUrl("https://api.example.com")
    .filter(addAuthHeader)
    .filter(logResponse)
    .build();
```

Every request made through this `webClient` now automatically gets the
auth header and logs the response status — no need to repeat that logic
at every single call site.

## Why It Matters

`WebClient` filters gather up cross-cutting HTTP client concerns (auth,
logging, retries, metrics) in one place, applied consistently to every
outgoing call from a shared client — the same "cross-cutting concerns"
idea `WebFilter` applies to incoming requests, just for outgoing ones.
