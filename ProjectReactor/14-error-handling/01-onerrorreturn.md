# onErrorReturn()

## In Simple Terms

`.onErrorReturn(fallbackValue)` catches an error and replaces it with a single,
**static fallback value**, letting the stream complete successfully instead of
propagating the failure. It's the simplest way to say "if this fails, just use this
default instead."

## Simple Example

```java
Mono<Integer> result = Mono.just("abc")
    .map(Integer::parseInt) // throws NumberFormatException
    .onErrorReturn(-1);

result.subscribe(value -> System.out.println("Result: " + value));
// Result: -1
```

You can also match specific exception types:

```java
Flux.just("1", "2", "notanumber", "4")
    .map(Integer::parseInt)
    .onErrorReturn(NumberFormatException.class, -1)
    .subscribe(n -> System.out.println("Got: " + n));
// Got: 1
// Got: 2
// Got: -1   (stream ENDS here - onErrorReturn still terminates the sequence)
```

**Important:** `.onErrorReturn()` still ends the stream after emitting the fallback
— it doesn't let the original sequence continue from where it left off (notice `4`
never appears above).

## Why It Matters

`.onErrorReturn()` is the simplest error-recovery tool, perfect for cases where a
fixed default value makes sense on failure — e.g., returning `0` for a failed
calculation, or an empty result placeholder — without needing any additional async
logic.
