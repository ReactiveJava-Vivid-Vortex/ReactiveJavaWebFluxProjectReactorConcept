# onErrorReturn()

## In Simple Terms

`.onErrorReturn()` catches an error and swaps it out for one fixed backup
value, letting the stream wrap up as if nothing went wrong. It's the
simplest way to say "if this breaks, just use this instead."

## Simple Example

```java
Mono<Integer> result = Mono.just("abc")
    .map(Integer::parseInt) // throws NumberFormatException
    .onErrorReturn(-1);

result.subscribe(value -> System.out.println("Result: " + value));
// Result: -1
```

You can also target a specific kind of exception:

```java
Flux.just("1", "2", "notanumber", "4")
    .map(Integer::parseInt)
    .onErrorReturn(NumberFormatException.class, -1)
    .subscribe(n -> System.out.println("Got: " + n));
// Got: 1
// Got: 2
// Got: -1   (stream ENDS here - onErrorReturn still terminates the sequence)
```

**Good to know:** `.onErrorReturn()` still ends the stream right after
handing out the fallback — it doesn't let the rest of the original sequence
continue (notice `4` never shows up above).

## Why It Matters

`.onErrorReturn()` is the simplest recovery tool around, perfect for when a
fixed default makes sense on failure — returning `0` for a failed
calculation, or a placeholder result — without needing any extra logic.
