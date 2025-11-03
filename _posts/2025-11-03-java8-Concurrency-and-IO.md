# Java Concurrency & IO (Java 8, 17 & 20) – In-Depth Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

---
## Table of Contents
1. [Module 1 – Basics & Theory](#module-1--basics--theory)
2. [Module 2 – In-Depth Topics](#module-2--in-depth-topics)
3. [Module 3 – Multiple Choice Questions (MCQs)](#module-3--multiple-choice-questions-mcqs)
4. [Module 4 – Practical Hands-On Exercises](#module-4--practical-hands-on-exercises)
5. [Module 5 – Common Compilation & Runtime Errors](#module-5--common-compilation--runtime-errors)
6. [Module 6 – Techniques, Patterns & Hacks](#module-6--techniques-patterns--hacks)
7. [Module 7 – Interview Questions & Answers](#module-7--interview-questions--answers)
8. [Module 8 – Solutions Appendix](#module-8--solutions-appendix)

---
<div style="page-break-before: always;"></div>

# Module 1 – Basics & Theory

### 1.1 Concurrency Evolution Overview (Java 8 → 17 → 20)
- Java 8: Fork/Join maturity, parallel streams, CompletableFuture API, NIO.2 file operations, AsynchronousChannel.
- Java 17 (LTS): Improvements in performance, sealed classes (modeling concurrent hierarchies), enhanced pseudo-random generators (thread-safe design), refined memory model clarifications.
- Java 19/20 (Preview → Incubating → evolving): Virtual Threads (Project Loom), Structured Concurrency (Incubator), Scoped Values. (Java 20 continued previews.)

### 1.2 Threading Models
| Model | Characteristics | Pros | Cons |
|-------|-----------------|------|------|
| Platform Threads | Mapped 1:1 to OS threads | Mature, predictable | Limited scalability (blocking costs) |
| Virtual Threads (Loom) | Lightweight, many per OS thread, scheduled by JDK | Massive concurrency, simpler blocking semantics | Still evolving (debugger tooling differences) |
| Fork/Join Pool | Work-stealing for task splitting | Efficient parallel divide & conquer | Overhead for fine-grained tasks |
| Reactive (CompletableFuture + async IO) | Non-blocking composition | Resource efficient | Complexity in error flow |

### 1.3 ASCII Diagram: Virtual Threads vs Platform Threads
```
Platform Threads (Few -> Many tasks queued)
+-----------+   +-----------+   +-----------+
|   OS T1   |   |   OS T2   |   |   OS T3   |
+-----+-----+   +-----+-----+   +-----+-----+
      |                |                |
   Tasks (queued)   Tasks (queued)   Tasks (queued)

Virtual Threads (Many -> Few carriers)
+--------------------------------------------------+
|          Carrier OS Thread (T1)                  |
|  v-thread-1  v-thread-2  v-thread-3  v-thread-n  |
+--------------------------------------------------+
|          Carrier OS Thread (T2)                  |
|  v-thread-X  v-thread-Y  ...                     |
+--------------------------------------------------+
```

### 1.4 Core Concurrency Concepts
- Thread safety: data race avoidance (atomicity, visibility, ordering).
- Memory Model: `volatile` ensures visibility & ordering; synchronized blocks provide mutual exclusion + happens-before edges.
- Immutability as concurrency strategy.

### 1.5 Executors & Pools
```java
ExecutorService pool = Executors.newFixedThreadPool(8);
Future<Integer> future = pool.submit(() -> compute());
Integer result = future.get();
```
Prefer `try (ExecutorService pool = ...)` with custom wrapper or ensure shutdown.

### 1.6 CompletableFuture Basics (Java 8)
```java
CompletableFuture<Integer> cf = CompletableFuture.supplyAsync(() -> 42)
    .thenApply(n -> n * 2)
    .thenCombine(CompletableFuture.supplyAsync(() -> 10), Integer::sum);
int val = cf.join(); // 94
```

### 1.7 NIO.2 File Operations
```java
Path p = Path.of("data.txt");
List<String> lines = Files.readAllLines(p, StandardCharsets.UTF_8);
Files.write(Path.of("out.txt"), lines);
```

### 1.8 Asynchronous Channels
```java
AsynchronousFileChannel ch = AsynchronousFileChannel.open(p, StandardOpenOption.READ);
ByteBuffer buf = ByteBuffer.allocate(1024);
ch.read(buf, 0, buf, new CompletionHandler<Integer, ByteBuffer>() {
    public void completed(Integer bytes, ByteBuffer att){ System.out.println("Read: " + bytes); }
    public void failed(Throwable ex, ByteBuffer att){ ex.printStackTrace(); }
});
```

### 1.9 Virtual Threads (Java 19/20 Preview)
```java
try (ExecutorService vt = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<String> f = vt.submit(() -> Thread.currentThread() + " working");
    System.out.println(f.get());
}
```

### 1.10 Structured Concurrency (Incubator)
Goal: Treat multiple tasks as a single unit of work with lifecycle scoping.
```java
// Pseudocode: StructuredTaskScope (Java 19/20 preview)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser());
    Future<List<String>> orders = scope.fork(() -> fetchOrders());
    scope.join(); // wait
    scope.throwIfFailed();
    return user.result() + orders.result();
}
```

### 1.11 Scoped Values (Preview)
Thread-local alternative with immutable data propagation for virtual threads.

### 1.12 Exercise Set (Basics)
1. Launch 10 virtual threads printing id.
2. Use CompletableFuture to fetch two numbers and combine results.
3. Read a file lines count using NIO.2.
4. Implement simple structured concurrency snippet.

(See Module 8 for solutions.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Synchronization & Happens-Before
Happens-before edges established by:
- Lock release -> subsequent lock acquire.
- Write to volatile -> subsequent read of same volatile.
- Thread start/join boundaries.

### 2.2 Atomic Classes vs Locks
| Feature | Atomic (e.g., AtomicInteger) | Lock/Synchronized |
|---------|------------------------------|------------------|
| Granularity | Single variable | Arbitrary critical section |
| Blocking | Non-blocking (CAS) | Blocking | 
| Composability | Hard (ABA issues) | Easier (coarse-grained) |
| Performance | High for low contention | Varies with contention |

```java
AtomicInteger counter = new AtomicInteger();
IntStream.range(0, 1_000_000).parallel().forEach(i -> counter.incrementAndGet());
```

### 2.3 Lock Types
- Intrinsic lock (synchronized)
- ReentrantLock
- ReadWriteLock
- StampedLock (Java 8) for optimistic reads

StampedLock example:
```java
class Point {
    private double x,y; private final StampedLock lock = new StampedLock();
    double distanceFromOrigin(){
        long stamp = lock.tryOptimisticRead();
        double cx = x, cy = y;
        if(!lock.validate(stamp)){
            stamp = lock.readLock();
            try { cx = x; cy = y; } finally { lock.unlockRead(stamp); }
        }
        return Math.hypot(cx, cy);
    }
    void move(double dx,double dy){ long stamp = lock.writeLock(); try { x+=dx; y+=dy; } finally { lock.unlockWrite(stamp);} }
}
```

### 2.4 Deadlock Patterns & Avoidance
Avoid cyclic lock ordering, hold locks briefly, prefer higher-level constructs (concurrent collections, atomic variables). Use thread dumps (jcmd, jstack) for diagnosis.

### 2.5 CompletableFuture Advanced Patterns
```java
CompletableFuture<Integer> a = CompletableFuture.supplyAsync(() -> slowA());
CompletableFuture<Integer> b = CompletableFuture.supplyAsync(() -> slowB());
CompletableFuture<Integer> fastest = a.applyToEither(b, x -> x);
CompletableFuture<Integer> combined = a.thenCompose(x -> fetchDependent(x));
```
Exception handling:
```java
a.exceptionally(ex -> { log(ex); return -1; });
```

### 2.6 Parallel Streams vs Explicit Async
Parallel streams suited to stateless transformations over data sets; CompletableFuture for asynchronous combinations across IO-bound boundaries.

### 2.7 Virtual Threads Deep Dive
- Blocking operations (e.g., socket read) park virtual thread without tying OS thread.
- Scheduler uses continuation to resume.
- Avoid thread-local heavy reliance; use Scoped Values.

### 2.8 Structured Concurrency Error Propagation
Scopes capture failures; `ShutdownOnFailure` cancels siblings. Simplifies cancellation logic compared to manual Future tracking.

### 2.9 NIO Channels & Buffers
Non-blocking selectors for scalable network IO; high-performance file operations using `FileChannel.transferTo/transferFrom`.

### 2.10 Zero-Copy File Transfer
```java
try (FileChannel src = FileChannel.open(Path.of("big.dat"));
     FileChannel dst = FileChannel.open(Path.of("copy.dat"), StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    long pos = 0, size = src.size();
    while (pos < size) { pos += src.transferTo(pos, size-pos, dst); }
}
```

### 2.11 AsynchronousFileChannel Write
```java
ByteBuffer data = ByteBuffer.wrap("Hello".getBytes());
ch.write(data, 0, data, new CompletionHandler<Integer, ByteBuffer>() {
    public void completed(Integer written, ByteBuffer buf){ System.out.println("Written: " + written); }
    public void failed(Throwable ex, ByteBuffer buf){ ex.printStackTrace(); }
});
```

### 2.12 JDK 17 Additions Relevant to Concurrency
- Improved performance of G1/ZGC (affects pause times)
- Foreign Memory API still incubating (in later versions) enabling safer off-heap interactions (affects IO use cases)

### 2.13 Loom Impact on Design
Simplify reactive complexity: revert callback pyramids to sequential style while retaining scalability.

### 2.14 Exercise Set (In-Depth)
1. Implement StampedLock protected coordinate update and read.
2. Compose two CompletableFutures with fallback on exception.
3. Convert callback-style async to structured concurrency snippet.
4. Build file copy utility using zero-copy API.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Which lock supports optimistic read? | ReentrantLock | StampedLock | ReadWriteLock | Intrinsic | B | StampedLock offers optimistic read via stamp validation. |
| 2 | `CompletableFuture.join()` differs from `get()` by? | Ignores exceptions | Wraps exceptions in CompletionException | Blocks forever | Non-blocking | B | join throws unchecked CompletionException wrapping cause. |
| 3 | Virtual threads primarily improve? | CPU cache | Thread-local speed | Blocking scalability | GC throughput | C | They allow many blocking tasks cheaply. |
| 4 | Deadlock avoidance technique? | Increase thread count | Random lock ordering | Consistent lock ordering | Use sleep loops | C | Consistent ordering prevents cycles. |
| 5 | Best fit for combining two async calls? | Parallel Stream | CompletableFuture | synchronized | AtomicInteger | B | CompletableFuture composes async tasks. |
| 6 | Zero-copy file transfer method? | `Files.copy()` | `transferTo/transferFrom` | `readAllBytes` | `BufferedReader.readLine` | B | FileChannel transfer methods utilize OS features. |
| 7 | Optimistic read fallback uses? | validate() | revalidate() | rollback() | compare() | A | `validate(stamp)` checks validity. |
| 8 | Structured concurrency scope purpose? | Random scheduling | Manage thread priorities | Treat tasks as single unit | Replace executor service | C | Encapsulates tasks lifecycle. |
| 9 | `volatile` guarantees? | Atomic increment | Mutual exclusion | Visibility and ordering | Lock freeing | C | Ensures visibility and ordering, not atomic compound ops. |
| 10 | Virtual thread creation API? | `new Thread()` | `Executors.newVirtualThreadPerTaskExecutor()` | `Thread.startVirtual()` | `VirtualThread.new()` | B | Provided via Executors factory. |
| 11 | Async file IO callback interface? | `Runnable` | `CompletionHandler` | `Callable` | `Handler` | B | `CompletionHandler` used. |
| 12 | CAS stands for? | Concurrent Access Semaphore | Compare And Swap | Conditional Atomic Section | Control And Sync | B | Atomic hardware primitive. |
| 13 | Which is stateful high-level concurrency builder? | `StampedLock` | `StructuredTaskScope` | `AtomicBoolean` | `ThreadLocal` | B | StructuredTaskScope groups tasks. |
| 14 | Which improves blocking scalability without rewriting code? | Virtual Threads | synchronized | ThreadLocal | Fork/Join only | A | Virtual threads simplify scalability. |
| 15 | `CompletableFuture.exceptionally` used for? | Cancellation | Logging only | Provide fallback value | Retrying automatically | C | Supplies fallback value on exception.

### MCQ Extension Exercise
Add 2 MCQs: one on Scoped Values, one on differences between platform & virtual threads scheduling.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Virtual Thread Burst
Create 1,000 virtual threads performing simulated IO (sleep 10ms) and collect results.

### Exercise 4.2: CompletableFuture Aggregation
Fetch three values asynchronously; compute combined metric; handle one failure gracefully.

### Exercise 4.3: StampedLock Coordinates
Implement thread-safe point with optimistic read and write lock.

### Exercise 4.4: Zero-Copy File Copy
Copy large file using `transferTo` measuring time.

### Exercise 4.5: Structured Concurrency Retrieval
Fetch user, orders, recommendations concurrently; cancel siblings on failure.

### Exercise 4.6: Async File Channel Read & Process
Read file chunk asynchronously and process lines.

### Exercise 4.7: Volatile Flag Stop
Implement worker loop responding to volatile stop flag.

### Exercise 4.8: Atomic Counter Benchmark
Compare synchronized vs AtomicInteger increment performance.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Forgetting `shutdown()` on Executor
Resource leak; tasks may keep JVM alive.

### 5.2 Blocking Call Inside Parallel Stream
Potential thread starvation if blocking IO inside common ForkJoinPool.

### 5.3 Deadlock Example
```java
Object a = new Object(); Object b = new Object();
Thread t1 = new Thread(() -> { synchronized(a){ sleep(50); synchronized(b){ } } });
Thread t2 = new Thread(() -> { synchronized(b){ sleep(50); synchronized(a){ } } });
t1.start(); t2.start();
```

### 5.4 Missed Volatile
```java
class Runner { boolean running = true; void stop(){ running=false; } }
// Without volatile or synchronization other threads may not see update.
```

### 5.5 Incorrect Atomic Compound
```java
AtomicInteger ai = new AtomicInteger(0);
if(ai.get() == 0){ ai.incrementAndGet(); } // Race: two threads may observe 0
```
Fix: `ai.compareAndSet(0,1);`

### 5.6 Virtual Thread Misuse (Busy Spin)
Busy loops waste carrier thread time; yield or blocking operations preferred.

### 5.7 StampedLock Stamp Mismanagement
Failing to release write lock or using invalid stamp causes IllegalMonitorStateException.

### 5.8 CompletableFuture Swallowing Exception
`get()` wraps exception; ignoring cause reduces diagnostics.

### 5.9 Exercise: Identify 3 Issues
```java
ExecutorService ex = Executors.newFixedThreadPool(4);
ex.submit(() -> Thread.sleep(500));
CompletableFuture<Void> cf = CompletableFuture.runAsync(() -> { throw new RuntimeException("boom"); });
AtomicInteger c = new AtomicInteger();
if(c.get() == 0) c.incrementAndGet();
```

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Bulk Virtual Thread Launcher Utility
```java
static List<Future<?>> launch(int n, Callable<?> task){
    try (ExecutorService vt = Executors.newVirtualThreadPerTaskExecutor()) {
        return IntStream.range(0,n).mapToObj(i -> vt.submit(task)).toList();
    }
}
```

### 6.2 Structured Concurrency Wrapper
Combine tasks with simpler cancellation semantics.

### 6.3 Time-Bounded Operations
Use `Future.get(timeout)` or scope shutdown on failure.

### 6.4 Semaphore Throttling IO
```java
Semaphore sem = new Semaphore(10);
void fetch(){ try { sem.acquire(); /* IO */ } catch(InterruptedException e){ Thread.currentThread().interrupt(); } finally { sem.release(); } }
```

### 6.5 Bulkhead Pattern
Separate thread pools per subsystem to isolate failures.

### 6.6 Circuit Breaker Sketch
Use atomic state & timestamps for open/close transitions.

### 6.7 Rate Limiting with ScheduledExecutor
```java
ScheduledExecutorService ses = Executors.newSingleThreadScheduledExecutor();
ses.scheduleAtFixedRate(() -> perform(), 0, 100, TimeUnit.MILLISECONDS);
```

### 6.8 IO Buffer Reuse
Reuse direct ByteBuffers for high-throughput network copy.

### 6.9 AtomicFieldUpdater for Low Overhead
```java
class Status { volatile int state; static final AtomicIntegerFieldUpdater<Status> UPD = AtomicIntegerFieldUpdater.newUpdater(Status.class, "state"); }
```

### 6.10 Exercise: Implement simple circuit breaker using AtomicLong and AtomicInteger.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Difference between `synchronized` and `ReentrantLock`?  
Lock offers tryLock, fairness, interruptible waits; synchronized simpler with implicit release.
2. Why virtual threads matter?  
Enable thousands of blocking operations concurrently with low resource footprint.
3. What is Structured Concurrency?  
API controlling lifetime of child tasks as a cohesive unit improving cancellation and error handling.
4. Compare CompletableFuture vs Future.  
CompletableFuture adds completion callbacks, chaining, combination; Future only blocking retrieval.
5. When to use StampedLock?  
High read concurrency with occasional writes requiring optimistic read path.
6. Drawbacks of naive parallel streams for IO?  
Blocking tasks consume ForkJoin worker threads leading to starvation.
7. How does volatile differ from Atomic?  
Volatile ensures visibility not atomic compound operations; Atomic provides atomic updates (CAS loops).
8. Best practices handling exceptions in async pipelines?  
Use exceptionally/handle, central logging, propagate meaningful fallback values.
9. Why zero-copy important?  
Reduces CPU overhead and memory copies between kernel/user space.
10. How do Scoped Values differ from ThreadLocal?  
Immutable, explicit scoping, cheap with virtual threads; ThreadLocal mutable & potential leaks.

### Exercise: Explain difference between platform and virtual thread scheduling.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. Virtual threads:
```java
try (ExecutorService vt = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0,10).forEach(i -> vt.submit(() -> System.out.println(Thread.currentThread())));
}
```
2. CompletableFuture combine:
```java
int result = CompletableFuture.supplyAsync(() -> 10)
    .thenCombine(CompletableFuture.supplyAsync(() -> 5), Integer::sum)
    .join();
```
3. File lines count:
```java
long count = Files.lines(Path.of("data.txt")).count();
```
4. Structured concurrency pseudocode shown earlier.

## Solutions: Module 2 Exercises
1. StampedLock usage given in example.
2. Fallback:
```java
int val = CompletableFuture.supplyAsync(() -> risky())
    .exceptionally(ex -> -1)
    .join();
```
3. Convert callback to structured concurrency using scope snippet.
4. Zero-copy file copy snippet in 2.10.

## Solutions: Module 3 MCQ Extension
1. Scoped Values MCQ sample answer: Provide immutably scoped data for virtual threads.
2. Scheduling difference MCQ sample answer: Virtual threads multiplex tasks on carrier threads vs platform one-to-one mapping.

## Solutions: Module 4 Selected Exercises
Exercise 4.2:
```java
CompletableFuture<Integer> a = CompletableFuture.supplyAsync(() -> slowA());
CompletableFuture<Integer> b = CompletableFuture.supplyAsync(() -> slowB());
CompletableFuture<Integer> c = CompletableFuture.supplyAsync(() -> slowC());
int combined = CompletableFuture.allOf(a,b,c)
    .thenApply(v -> a.join() + b.join() + c.join())
    .exceptionally(ex -> -1)
    .join();
```
Exercise 4.7:
```java
class Worker { volatile boolean running = true; void run(){ while(running){ doWork(); } } }
```
Exercise 4.8: Benchmark skeleton.
```java
long syncTime = time(() -> { int[] c = {0}; IntStream.range(0,N).parallel().forEach(i -> { synchronized(c){ c[0]++; } }); });
long atomicTime = time(() -> { AtomicInteger ai = new AtomicInteger(); IntStream.range(0,N).parallel().forEach(i -> ai.incrementAndGet()); });
```

## Solutions: Module 5 Error Identification (5.9)
Issues:
1. Not shutting down executor.
2. Ignoring InterruptedException in sleep (should handle). Also blocking inside pool thread.
3. CompletableFuture exception not handled.
4. Atomic compound non-atomic (race) – use compareAndSet.

## Solutions: Module 6 Circuit Breaker Exercise (Sketch)
```java
class Circuit {
    private final AtomicInteger failures = new AtomicInteger();
    private final AtomicLong openedAt = new AtomicLong(0);
    boolean call(Supplier<String> op){
        if(isOpen()){ return false; }
        try { op.get(); failures.set(0); return true; }
        catch(Exception e){ if(failures.incrementAndGet() >= 3){ openedAt.set(System.currentTimeMillis()); } return false; }
    }
    boolean isOpen(){ long openTime = openedAt.get(); return openTime != 0 && System.currentTimeMillis() - openTime < 10_000; }
}
```

## Solutions: Module 7 Exercise
Platform threads map 1:1 to OS threads; scheduler is OS. Virtual threads are user-mode constructs scheduled by JVM; blocking operations unmount virtual thread from carrier allowing reuse.

---

### End Notes
Adopt virtual threads for massive IO-bound concurrency with simpler code. Continue using proven primitives for shared mutable state and measure before replacing tuned parallel strategies.
