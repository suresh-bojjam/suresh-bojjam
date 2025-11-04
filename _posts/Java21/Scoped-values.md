# Scoped Values (Java 21) – Comprehensive Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

Scoped values (JEP 446) provide a safe, structured alternative to thread-local variables, enabling immutable, efficiently inheritable context data bound to a lexical scope, especially effective with virtual threads and structured concurrency.

---
## Table of Contents
1. Module 1 – Basics & Theory
2. Module 2 – In-Depth Topics
3. Module 3 – Multiple Choice Questions (MCQs)
4. Module 4 – Practical Learning Examples
5. Module 5 – Compilation & Runtime Errors
6. Module 6 – Techniques & Hacks
7. Module 7 – Interview Questions
8. Module 8 – Solutions Appendix

---
<div style="page-break-before: always;"></div>

# Module 1 – Basics & Theory

### 1.1 Motivation
`ThreadLocal` has pitfalls: memory leaks (forgetting to remove), mutable state leading to race-like bugs, poor suitability for massive virtual thread counts. Scoped values offer:
- Lexical scoping (value only visible within bound block).
- Immutability (cannot mutate after binding).
- Efficient inheritance across child tasks.

### 1.2 Core Concepts
| Concept | Description |
|---------|-------------|
| ScopedValue<T> | Holder enabling binding of a value to the current thread within a lexical scope |
| `where(...)` | Binds one or multiple scoped values for the duration of a runnable/callable |
| Immutability | Bound value cannot be changed until scope exits |
| Nesting | Nested scopes can shadow outer bindings |

### 1.3 Basic Declaration & Binding
```java
import java.lang.ScopedValue;

static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

void handle() {
    ScopedValue.where(REQUEST_ID, "req-123", () -> {
        System.out.println("Handling " + REQUEST_ID.get());
    });
}
```

### 1.4 ASCII Diagram – Lexical Binding
```
+---------------------------+
| Outer Code               |
|  REQUEST_ID (unbound)    |
+-------------+-------------+
              | where(REQUEST_ID,"abc")
              v
        +-------------------+
        | Inner Scope       |
        | REQUEST_ID = abc  |
        +---------+---------+
                  | exit scope -> unbound
```

### 1.5 Immutability & Shadowing
```java
ScopedValue.where(REQUEST_ID, "outer", () -> {
    ScopedValue.where(REQUEST_ID, "inner", () -> {
        System.out.println(REQUEST_ID.get()); // inner
    });
    System.out.println(REQUEST_ID.get()); // outer
});
```

### 1.6 Multiple Bindings
```java
static final ScopedValue<String> USER = ScopedValue.newInstance();
static final ScopedValue<String> LOCALE = ScopedValue.newInstance();

ScopedValue.where(USER, "alice", LOCALE, "en-US", () -> {
    System.out.println(USER.get() + " / " + LOCALE.get());
});
```

### 1.7 Access Outside Scope
Calling `REQUEST_ID.get()` when unbound throws `IllegalStateException` (or returns absent — per final spec behavior: throws).

### 1.8 Exercise Set (Basics)
1. Bind a request id and print inside scope.
2. Nest scopes showing shadowing.
3. Bind two scoped values simultaneously.
4. Attempt access outside scope and observe handling.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Differences vs ThreadLocal
| Aspect | ThreadLocal | ScopedValue |
|--------|-------------|-------------|
| Mutability | Mutable | Immutable after binding |
| Lifecycle | Manual set/remove | Automatic by lexical scope |
| Inheritance Model | Copies on child thread creation | Efficient for virtual threads, lexical binding propagation |
| Leak Risk | High if not removed | Low (scope boundaries) |
| API Complexity | Requires discipline | Simpler, structured |

### 2.2 Propagation with Virtual Threads
Virtual threads started within a scope see bindings automatically; no manual copying required.

### 2.3 Structured Concurrency Integration
```java
ScopedValue.where(REQUEST_ID, "r-77", () -> {
    try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
        scope.fork(() -> doWork(REQUEST_ID.get()));
        scope.join();
    }
});
```

### 2.4 Shadowing Depth
Nested binding only visible inside its block; outer binding resumes afterwards.

### 2.5 Performance Considerations
Scoped values implement efficient stack-like structure; reads are O(1) with minimal overhead compared to thread-local map lookups.

### 2.6 Error Handling Strategy
Scopes naturally ensure cleanup; exceptions do not require manual removal.

### 2.7 Security / Isolation
Reduces accidental cross-request data leakage by scoping context strictly to lexical block.

### 2.8 Migrating from ThreadLocal
Pattern:
```java
// Before
static final ThreadLocal<String> REQUEST_ID = new ThreadLocal<>();
void handle(){
    REQUEST_ID.set("r");
    try { process(); } finally { REQUEST_ID.remove(); }
}
// After
ScopedValue.where(REQUEST_ID, "r", () -> process());
```

### 2.9 Multiple Bindings & Ordering
Binding list pairs values sequentially; all active for duration. No partial binding reverts until exit.

### 2.10 Debugging Unbound Access
Wrap `get()` in a helper catching `IllegalStateException` to improve diagnostic message.

### 2.11 Exercise Set (In-Depth)
1. Migrate ThreadLocal usage to scoped value.
2. Demonstrate virtual thread inheriting binding.
3. Use structured scope with binding.
4. Implement helper that returns optional-like for binding presence.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Scoped values primarily solve? | GC tuning | Safer context passing | Faster serialization | UI rendering | B | Provide structured context. |
| 2 | Value mutability after `where`? | Mutable | Immutable | Depends on thread | Auto-random | B | Binding is immutable. |
| 3 | Unbound `get()` leads to? | Null | Empty Optional | IllegalStateException | 0 | C | Access outside binding throws. |
| 4 | Shadowing effect? | Overwrites permanently | Temporary override inside inner scope | Deletes outer binding | Causes error | B | Inner scope shadow only. |
| 5 | Virtual thread propagation? | Manual copy | Automatic inheritance | No inheritance | Requires wrapper | B | Seamless propagation. |
| 6 | ThreadLocal leak risk vs ScopedValue? | Higher | Lower | Same | Unknown | A (ThreadLocal higher) / Actually ScopedValue lower | A | ThreadLocal prone to leaks. |
| 7 | Binding multiple values? | Not possible | Use multiple `where` calls only | Single `where` with pairs | Requires builder pattern | C | API supports multiple pairs. |
| 8 | Performance read complexity? | O(n) chain | O(1) | O(log n) | Exponential | B | Efficient constant-time. |
| 9 | Cleanup required? | Manual remove | Automatic at scope exit | Requires GC finalize | OS signal | B | Lexical scope handles. |
| 10 | Migration benefit? | More verbose code | Less boilerplate & leak risk | Slower operations | Harder debugging | B | Simplifies cleanup. |
| 11 | Structured concurrency synergy? | None | Enhanced, binds context to tasks | Slows tasks | Breaks API | B | Combined scoping fits task lifecycle. |
| 12 | Access pattern outside scope? | Returns default | Throws error | Creates new binding | Ignores | B | Throws. |
| 13 | Creating instance? | `new ScopedValue<>()` | `ScopedValue.newInstance()` | `ScopedValue.of()` | `ScopedValue.create()` | B | Factory method. |
| 14 | Overriding outer binding permanently? | Yes | No | Only if same value | Only on main thread | B | Shadow ends on exit. |
| 15 | Using for security? | Helps prevent cross-request data leak | Increases leak risk | No effect | Disables TLS | A | Scoping isolates context.

### MCQ Extension
16. Migration pattern from ThreadLocal replaces try-finally with? (A: lexical `where` block)  
17. Reads outside binding produce? (A: IllegalStateException)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Basic Binding & Access
```java
static final ScopedValue<String> RID = ScopedValue.newInstance();
ScopedValue.where(RID, "X-1", () -> {
    System.out.println(RID.get());
});
```

### 4.2 Nested Shadow
```java
ScopedValue.where(RID, "outer", () -> {
    ScopedValue.where(RID, "inner", () -> System.out.println(RID.get()));
    System.out.println(RID.get()); // outer
});
```

### 4.3 Multiple Bindings
```java
static final ScopedValue<String> USER = ScopedValue.newInstance();
static final ScopedValue<String> ROLE = ScopedValue.newInstance();
ScopedValue.where(USER, "tom", ROLE, "admin", () -> {
    System.out.println(USER.get() + ":" + ROLE.get());
});
```

### 4.4 Virtual Thread Propagation
```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {
    ScopedValue.where(USER, "eva", () -> {
        exec.submit(() -> System.out.println(USER.get())); // inherits
    });
}
```

### 4.5 Structured Scope Example
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    ScopedValue.where(RID, "req-77", () -> {
        var a = scope.fork(() -> fetchA(RID.get()));
        var b = scope.fork(() -> fetchB(RID.get()));
        scope.join(); scope.throwIfFailed();
        System.out.println(a.get() + b.get());
    });
}
```

### 4.6 Migration from ThreadLocal
Already shown; implement using where instead of set/remove.

### 4.7 Unbound Access Demonstration
```java
static void test(){
    System.out.println(RID.get()); // throws IllegalStateException
}
```

### 4.8 Exercise Set (Practical)
1. Shadow outer RID inside nested scope.
2. Bind USER & ROLE and print.
3. Demonstrate virtual thread inheritance.
4. Structured scope retrieval of RID in tasks.
5. Show exception on unbound access.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Unbound Access
`IllegalStateException` thrown when calling `get()` outside scope.

### 5.2 Incorrect Factory Use
Trying `new ScopedValue<>()` -> does not compile (constructor not public?). Must use `newInstance()`.

### 5.3 Shadow Logic Misunderstanding
Expecting inner value after block -> results in outer; not an error but logic bug.

### 5.4 Passing Null Value
Binding `null` allowed; handle carefully to avoid NPE later; design optional wrappers.

### 5.5 ThreadLocal Confusion
Mixing scoped value & ThreadLocal for same context key causing inconsistent state.

### 5.6 Exercise Set (Errors)
Identify: unbound access, wrong construction, mixing context mechanisms.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Context Bundle Class
Group related scoped values into a context provider.

### 6.2 Diagnostic Wrapper
Provide helper: `String safeRid(){ return RID.isBound()?RID.get():"<none>"; }` (conceptual `isBound()` if added; else use try-catch).

### 6.3 Layered Scoping
Bind request-level values outer; user/session inner; avoid overshadow confusion.

### 6.4 Minimal Surface
Use scoped values only for truly request-scope immutable data (IDs, auth context), not mutable caches.

### 6.5 Testing Patterns
Unit test with explicit `where` binding blocks; avoid hidden global state.

### 6.6 Observability
Inject tracing ID via scoped value; access seamlessly in deep call stack without parameter plumbing.

### 6.7 Fallback Emulation
Wrap get in optional-like pattern to avoid try/catch noise.

### 6.8 Migration Steps Checklist
Identify ThreadLocal usages -> categorize -> convert simple ones first -> remove manual finally cleanup.

### 6.9 Performance Microbenchmark
Compare `ThreadLocal.get()` vs `ScopedValue.get()` using JMH (setup suggestions).

### 6.10 Exercise Set (Techniques)
Design conversion plan for app with multiple ThreadLocals (auth, correlationId, locale) to scoped values.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. Why were scoped values introduced over ThreadLocal?
2. Describe propagation with virtual threads.
3. Explain immutability benefits.
4. How does shadowing work?
5. Migration strategy from ThreadLocal.
6. When to avoid scoped values.
7. Integration with structured concurrency.
8. Error handling on unbound access.
9. Security improvements.
10. Performance considerations.

### Exercise
Design a request processing pipeline using scoped values for tracing and locale, integrated with virtual threads.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
Binding, nested shadow, multi-binding, unbound access demonstration.

## Solutions: Module 2 Exercises
ThreadLocal migration snippet; virtual thread inheritance; structured scope retrieval; helper wrapper.

## Solutions: Module 3 MCQ Extension
16: Use lexical `where` block; 17: IllegalStateException on unbound read.

## Solutions: Module 4 Practical
Shadow outer; multi-binding; virtual thread example; structured scope tasks; unbound exception.

## Solutions: Module 5 Errors
Access outside scope throws; must use `newInstance()`; avoid mixing ThreadLocal & scoped value.

## Solutions: Module 6 Techniques
Conversion plan listing context keys; ensure immutability; remove manual cleanup.

## Solutions: Module 7 Exercise
Pipeline: outer scope binds RID & locale; inner forks tasks reading values; each log includes tracing ID automatically.

---

### End Notes
Scoped values advance context propagation: concise, safe, and well-aligned with virtual threads and structured concurrency. Adopt them to reduce boilerplate, prevent leaks, and clarify intent of immutable per-request data.
