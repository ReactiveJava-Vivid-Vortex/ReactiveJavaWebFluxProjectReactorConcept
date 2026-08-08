# Q1. What Is a Mono?

## Simple Explanation (Think of a Vending Machine Slot)

A `Mono<T>` is a slot in a vending machine that gives you **at most one item** —
never two, sometimes zero (out of stock), sometimes it jams (error).

```
Press the button (subscribe)
        │
   ┌────┴─────┬──────────────┐
   ▼           ▼              ▼
 Item drops  Nothing drops  Machine jams
 (VALUE)     (EMPTY)        (ERROR)
```

Those are the **only three things** that can ever happen with a `Mono` — there is
no fourth outcome, and it's not a coincidence: it's the universal Reactive Streams
signal grammar (`onSubscribe onNext* (onError | onComplete)?`) with `onNext`
capped at *at most once* instead of unlimited.

---

## Q2. What Are the Three Outcomes, Concretely?

```java
// Outcome 1: SUCCESS with a value
Mono.just("hello").subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e),
    () -> System.out.println("Complete")
);
// Value: hello
// Complete

// Outcome 2: EMPTY (success, no value — like "not found")
Mono.empty().subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e),
    () -> System.out.println("Complete")
);
// Complete   (no "Value:" line!)

// Outcome 3: ERROR
Mono.error(new RuntimeException("failed")).subscribe(
    v -> System.out.println("Value: " + v),
    e -> System.out.println("Error: " + e),
    () -> System.out.println("Complete")
);
// Error: failed   (no "Complete" line!)
```

| Outcome | Signal(s) | Real-World Meaning |
|---|---|---|
| Success + value | `onNext` then `onComplete` | "Here's your result" |
| Empty | `onComplete` only | "Valid lookup, nothing found" (not an error!) |
| Error | `onError` only | "Something went wrong" |

---

## Q3. What Are the Factory Methods, and When Do I Use Each?

| Factory | Use When... | Eager or Lazy? |
|---|---|---|
| `Mono.just(v)` | You already have a non-null value in hand | **Eager** |
| `Mono.empty()` | You want to represent "no value," successfully | — |
| `Mono.error(t)` | You want to represent a failure | — |
| `Mono.fromSupplier(fn)` | Lazy sync computation, no checked exceptions | Lazy |
| `Mono.fromCallable(fn)` | Lazy sync computation that MAY throw checked exceptions | Lazy |
| `Mono.fromRunnable(fn)` | A side effect with no return value (`Mono<Void>`) | Lazy |
| `Mono.defer(fn)` | Need a fresh, whole new Mono chosen per subscriber | Lazy |
| `Mono.create(sink -> ..)` | Bridging a legacy callback-based API | Manual |

**The biggest trap:** confusing eager `just()` with lazy `fromSupplier()`.

```java
// BAD: fetchFromDb() runs IMMEDIATELY, even with no subscriber!
Mono<User> bad = Mono.just(fetchFromDb());

// GOOD: fetchFromDb() only runs once someone subscribes
Mono<User> good = Mono.fromSupplier(() -> fetchFromDb());
```

---

## Q4. What Is `MonoSink`, and When Do I Need `Mono.create()`?

`MonoSink` is your manual "microphone" for bridging non-reactive, callback-based
APIs into a `Mono`. Call exactly one of `success()`, `success(value)`, or `error()`
— once.

```java
Mono<String> mono = Mono.create(sink -> {
    legacyAsyncCall(new LegacyCallback() {
        public void onSuccess(String result) { sink.success(result); }
        public void onFailure(Throwable error) { sink.error(error); }
    });
});
```

Use this **sparingly** — only for genuinely non-reactive sources. Everything else
should use one of the factory methods above.

---

## Q5. What Happens If the Mono Is Empty — How Do I Handle It?

```java
public Mono<User> findUser(String id) {
    return database.findById(id); // returns Mono.empty() if not found
}

// Option A: default value
findUser("x").defaultIfEmpty(User.GUEST).subscribe(System.out::println);

// Option B: switch to a different (possibly async) Mono
findUser("x").switchIfEmpty(backupDatabase.findById("x")).subscribe(System.out::println);

// Option C: treat empty as an error
findUser("x").switchIfEmpty(Mono.error(new UserNotFoundException("x"))).subscribe(...);
```

**Common beginner mistake:** treating "empty" and "error" as the same thing. They
are not — empty is a *valid, successful* outcome; error is a *failure*.

---

## Q6. How Many Ways Can I `.subscribe()` to a Mono?

```java
mono.subscribe();                                    // fire-and-forget
mono.subscribe(value -> handle(value));               // value only — errors silently logged by Reactor!
mono.subscribe(value -> handle(value), err -> handleError(err));       // value + error (safe minimum)
mono.subscribe(value -> handle(value), err -> handleError(err), () -> onDone()); // + completion
```

**Always supply an error consumer** unless you're absolutely sure the `Mono` can't
fail — otherwise failures are silently logged by Reactor internally instead of
being handled by your own application logic.

---

## Q7. Interview-Style Q&A

### Can a Mono emit `onNext` twice?

**No.** That would violate the contract — a `Mono` emits `onNext` at most once,
ever.

### Is `Mono.just(null)` valid?

**No** — throws `NullPointerException` immediately, because Reactive Streams
forbids `null` as a valid element. Use `Mono.justOrEmpty(possiblyNullValue)`
instead.

### Does `Mono.empty()` represent an error?

**No.** It's a successful, valid outcome — "nothing found," not "something broke."

### What's the difference between `Mono.fromSupplier()` and `Mono.fromCallable()`?

`fromSupplier()` is for code that can't throw a checked exception;
`fromCallable()` is for code that might (like file I/O) — both are equally lazy.

---

## Q8. Summary

| Outcome | Signals | Handled By |
|---|---|---|
| Success (value) | `onNext` + `onComplete` | `.map()`, normal chaining |
| Empty | `onComplete` only | `.switchIfEmpty()` / `.defaultIfEmpty()` |
| Error | `onError` only | `.onErrorResume()` / `.onErrorReturn()` |

| Concept | Key Takeaway |
|---|---|
| Mono | 0-or-1 async value — exactly 3 possible outcomes |
| Eager vs Lazy | `just()` runs immediately; `fromSupplier`/`fromCallable`/`defer` run on subscription |
| MonoSink | Manual bridge for legacy callback APIs — call success/error exactly once |
| Empty ≠ Error | Treat them as distinct — "not found" isn't automatically a failure |

### One sentence to remember

> **"A Mono is a vending machine slot: press the button and you get exactly one
> of three things — a snack, nothing, or a jam — never more, never a mixture."**
