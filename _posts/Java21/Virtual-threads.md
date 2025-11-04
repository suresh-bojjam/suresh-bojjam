# Virtual Threads (Java 21) – Comprehensive Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

Virtual Threads (JEP 436 finalization of Project Loom) dramatically increase concurrency scalability by decoupling Java threads from OS threads. They make blocking cheap and simplify high-throughput server code without complex async frameworks.

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
Traditional platform threads (OS-backed) are expensive: limited by memory stack size and context-switch overhead. Virtual threads provide lightweight, managed stacks enabling millions of concurrent operations using a blocking style that maps naturally to synchronous code.

### 1.2 Key Properties
| Property | Virtual Thread | Platform Thread |
|----------|----------------|-----------------|
| Creation Cost | Very low | Higher |
| Blocking Cost | Cheap (park/unpark) | Expensive (occupies OS thread) |
| Stack | Extensible, pageable | Fixed-size native stack |
| Intended Use | Massive concurrency (IO-bound) | Mixed workloads |
| Scheduling | User-mode continuation + carrier pool | OS scheduler |

### 1.3 Creating Virtual Threads
```java
Thread vt = Thread.ofVirtual().start(() -> System.out.println("Hello from VT"));
vt.join();
```

### 1.4 Executor for Virtual Threads
```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> fetchUser());
}
```

### 1.5 ASCII Architecture Diagram
```
+----------------------+    +------------------------+
| Virtual Thread (VT)  | -> | Carrier OS Thread Pool |
+----------+-----------+    +-----------+------------+
           |                              |
           | Parking/Continuation         |
           v                              v
     Suspended State                Active Execution
```

### 1.6 Parking & Continuations
When a virtual thread blocks (e.g., on I/O or `sleep`), its stack is captured and the carrier thread is released to run other tasks.

### 1.7 Exercise Set (Basics)
1. Start a virtual thread printing current thread type.
2. Submit 1000 tasks using virtual thread executor measuring total time.
3. Show blocking `Thread.sleep()` inside virtual thread.
4. Compare a platform vs virtual thread count creation limit.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Structured Concurrency Synergy
Virtual threads pair well with structured concurrency (JEP 453) for handling lifecycles.

### 2.2 Thread Factories
Custom naming:
```java
ThreadFactory tf = Thread.ofVirtual().name("vt-", 0).factory();
```

### 2.3 Bulk Submission Pattern
```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {
    List<Future<?>> futures = IntStream.range(0, 10_000)
        .mapToObj(i -> exec.submit(() -> serviceCall(i)))
        .toList();
    for (Future<?> f : futures) f.get();
}
```

### 2.4 I/O Blocking Semantics
Most blocking operations (network, file) will park virtual threads, not tie up carrier threads (subject to library implementation compliance).

### 2.5 Synchronization
`ReentrantLock`, monitors, and other sync primitives still work. Avoid pinning (cases preventing unmounting) such as holding monitors across blocking native calls.

### 2.6 Pinning
A virtual thread can become pinned if:
- Inside a native method.
- Holding a monitor while invoking a blocking native call.
During pinning, the carrier thread cannot switch tasks.

### 2.7 Debugging & Monitoring
Use `jcmd` or thread dumps—the number of virtual threads can be very large; dumps include identifiers.

### 2.8 Performance Considerations
- Prefer virtual threads for I/O-bound tasks.
- CPU-bound tasks may still require a fixed-size executor to avoid oversubscription.

### 2.9 Interop with Legacy APIs
Blocking JDBC calls become more scalable—though driver must not block at kernel level excessively.

### 2.10 Cancellation & Timeouts
Leverage `Future.cancel(true)` or cooperative interruption inside tasks.

### 2.11 Resource Management
Structured try-with-executors ensures tasks complete or cancel deterministically.

### 2.12 Migration Strategy
Replace complex async callback frameworks with straightforward synchronous code executed in virtual threads.

### 2.13 Exercise Set (In-Depth)
1. Create a named virtual thread factory and launch 100 tasks.
2. Demonstrate pinning scenario (simulate with blocking native sleep).
3. Measure latency of 10K trivial tasks.
4. Refactor callback-based code to virtual thread synchronous style.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Virtual threads target improvement in? | GPU usage | Massive concurrency scalability | Class loading speed | Serialization | B | Designed for many concurrent tasks. |
| 2 | Creating a virtual thread API call? | Thread.startVirtual() | Thread.ofVirtual() | new VirtualThread() | Executors.virtual() | B | Factory builder pattern. |
| 3 | Cheap blocking enabled by? | Kernel bypass | Stack capture & parking | JIT elimination | GC pause reduction | B | Continuations allow parking. |
| 4 | Executor for per-task virtual threads? | newCachedThreadPool | newVirtualThreadPerTaskExecutor | newFixedThreadPool | newWorkStealingPool | B | Dedicated Loom executor. |
| 5 | Pinning occurs when? | Arithmetic loop | Native blocking while holding monitor | Simple sleep | Logging | B | Blocks carrier thread. |
| 6 | Best workload for virtual threads? | Long CPU-bound tasks | Massive short I/O-bound tasks | GPU kernels | GUI rendering | B | Ideal for I/O-bound concurrency. |
| 7 | Monitoring large counts done with? | `top` | Thread dumps / jcmd | jmap only | JIT logs | B | Tools show virtual threads. |
| 8 | Structured concurrency benefit? | Random cancellation | Deterministic task lifecycle | Disables GC | Faster class loading | B | Manages scope & lifecycle. |
| 9 | Virtual thread stack characteristic? | Fixed large native stack | Extensible & pageable | No stack | Shared per process | B | Lightweight, grow-on-demand. |
| 10 | Oversubscription risk for? | I/O-bound tasks | CPU-bound tasks | Sleeping tasks | Parked tasks | B | Too many CPU-bound tasks degrade performance. |
| 11 | Interruption handling? | Same as platform threads | Not supported | Requires special API | Only via signals | A | Uses standard interruption semantics. |
| 12 | Pinning reduces? | Scalability | Memory safety | GC efficiency | Classpath size | A | Carrier thread cannot switch. |
| 13 | Legacy async migration approach? | Keep callbacks | Replace with synchronous tasks | Use signals | Fork processes | B | Simpler synchronous style. |
| 14 | Resource management pattern? | Random finalize | try-with executor | Shutdown hook only | Manual GC calls | B | try-with ensures closure. |
| 15 | Virtual threads vs Reactive frameworks? | Always slower | Eliminates need for scaling | Provide simpler synchronous alternative | Not comparable | C | Simplifies concurrency model.

### MCQ Extension
16. Carrier threads role? (A: Run virtual thread continuations)  
17. Using many CPU-heavy virtual threads causes? (A: Oversubscription & contention)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Launch Many Virtual Threads
```java
try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 1000).forEach(i -> exec.submit(() -> {
        Thread.sleep(10);
        return i;
    }));
}
```

### 4.2 Compare Platform vs Virtual Creation
```java
long start = System.nanoTime();
List<Thread> threads = new ArrayList<>();
for (int i=0;i<10_000;i++) threads.add(Thread.ofVirtual().unstarted(() -> {}));
threads.forEach(Thread::start);
for (Thread t : threads) t.join();
System.out.println("Virtual batch ms=" + (System.nanoTime()-start)/1_000_000);
```

### 4.3 Named Factory Usage
```java
ThreadFactory tf = Thread.ofVirtual().name("v-", 0).factory();
Thread t = tf.newThread(() -> System.out.println(Thread.currentThread()));
t.start();
t.join();
```

### 4.4 Pinning Demonstration (Conceptual)
```java
synchronized (lock) { // monitor held
    // Suppose call to blocking native I/O here -> potential pin
    Thread.sleep(50); // normal sleep usually unparks safely (not pinning)
}
```

### 4.5 Structured Concurrency Sample (Conceptual)
```java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Subtask<String> u = scope.fork(() -> fetchUser());
    Subtask<List<String>> o = scope.fork(() -> fetchOrders());
    scope.join();
    scope.throwIfFailed();
    return process(u.get(), o.get());
}
```

### 4.6 Simple Interruption
```java
Thread vt = Thread.ofVirtual().start(() -> {
    try { Thread.sleep(5_000); } catch (InterruptedException e) { System.out.println("Interrupted"); }
});
vt.interrupt();
vt.join();
```

### 4.7 Migration from Reactive Callback
Before (pseudo): callback chains -> After: synchronous blocking inside virtual thread performing sequential calls.

### 4.8 Exercise Set (Practical)
1. Measure time to create 50K virtual threads doing trivial sleep.
2. Show structured concurrency fetching two resources.
3. Simulate oversubscription with CPU-bound tasks and measure slowdown.
4. Implement interruption test.
5. Add custom naming pattern for audit logging.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Incorrect Factory Usage
Calling `new Thread()` expecting virtual semantics -> not virtual.

### 5.2 Pinning Causing Throughput Drop
Not a compilation error; runtime performance issue—carrier threads stuck.

### 5.3 InterruptedException Misuse
Ignoring interruption leads to unresponsive cancellation.

### 5.4 Resource Leak
Failing to close virtual thread executor -> tasks may continue unexpectedly.

### 5.5 IllegalState in Structured Scope
Accessing subtask result before `join()` -> IllegalStateException.

### 5.6 Exercise Set (Errors)
Identify: forgetting executor close, not handling InterruptedException, using platform threads inadvertently.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Concurrency Sizing
Allow virtually unlimited I/O tasks; bound CPU-intensive tasks with semaphore.

### 6.2 Adaptive Scheduling Strategy
Combine virtual thread executor for I/O + fixed pool for CPU.

### 6.3 Pinning Avoidance
Release locks before blocking calls; avoid long native method holds.

### 6.4 Cooperative Cancellation
Regularly check `Thread.interrupted()` in long-running loops.

### 6.5 Backpressure
Use rate limiter around virtual thread submissions to avoid overwhelming downstream services.

### 6.6 Observability
Tag thread names for tracing; integrate with structured concurrency scopes.

### 6.7 Error Aggregation
Use `ShutdownOnFailure` scopes to aggregate failures and cancel siblings.

### 6.8 Bulkhead Isolation
Separate executors for external APIs vs internal caches.

### 6.9 Warm-Up Strategy
Pre-run small tasks to prime JIT before large-scale launch.

### 6.10 Exercise Set (Techniques)
Design an architecture mixing virtual threads for API calls and a bounded pool for CPU-heavy parsing.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. How do virtual threads differ from platform threads internally?
2. Explain pinning and its effects.
3. Why does blocking become cheap with virtual threads?
4. When would you still choose a fixed thread pool?
5. Discuss migration from reactive frameworks.
6. How does structured concurrency complement virtual threads?
7. Strategies to avoid oversubscription.
8. Explain interruption handling with virtual threads.
9. What monitoring tools to observe virtual threads?
10. Provide use cases where virtual threads are suboptimal.

### Exercise
Design a high-throughput REST service architecture using virtual threads, structured scopes, and backpressure.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
Show thread printing name; executor with 1000 tasks; sleep example; compare creation scalability vs platform threads.

## Solutions: Module 2 Exercises
Named factory launching tasks; simulated pinning scenario; latency measurement process for 10K tasks; callback refactor demonstration.

## Solutions: Module 3 MCQ Extension
16: Carrier threads execute parked continuations; 17: Oversubscription leads to contention & slowdown.

## Solutions: Module 4 Practical
Timing 50K virtual sleeps; structured concurrency sample; CPU-bound overload example; interruption test; audit naming usage.

## Solutions: Module 5 Errors
Close executor in try-with; handle InterruptedException; ensure virtual creation via `Thread.ofVirtual()`.

## Solutions: Module 6 Techniques
Architecture mixing I/O virtual threads and CPU fixed pool with semaphore limiting.

## Solutions: Module 7 Exercise
REST design: virtual thread per request; structured scope for parallel data fetch; rate limiter for downstream calls; monitoring via thread dumps & metrics.

---

### End Notes
Virtual threads bring synchronous programming back to the forefront for scalable services. Embrace them for I/O-heavy workloads; remain mindful of pinning and oversubscription for optimal performance.
