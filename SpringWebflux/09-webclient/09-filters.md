# Filters (WebClient)

## In Simple Terms

`WebClient` supports **filters** (via `ExchangeFilterFunction`) — a way to intercept
and modify every outgoing request and incoming response, similar in spirit to
`WebFilter` for incoming server requests. Common uses include adding authentication
headers, logging, or retry logic uniformly across all calls made through a given
`WebClient` instance.

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

Every request made through this `webClient` instance now automatically includes the
auth header and logs the response status — no need to repeat that logic in every
individual call site.

## Why It Matters

`WebClient` filters centralize cross-cutting HTTP client concerns (auth, logging,
retries, metrics) in one place, applied consistently to every outgoing call made
through a shared client instance — the same "cross-cutting concerns" principle that
`WebFilter` applies to incoming requests, but for outgoing ones.
