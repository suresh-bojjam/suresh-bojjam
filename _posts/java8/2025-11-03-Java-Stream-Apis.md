# Java Stream APIs In-Depth Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

---
## Table of Contents
1. [Module 1 – Stream Basics](#module-1--stream-basics)
2. [Module 2 – In-Depth Stream Concepts](#module-2--in-depth-stream-concepts)
3. [Module 3 – Multiple Choice Questions (MCQs)](#module-3--multiple-choice-questions-mcqs)
4. [Module 4 – Practical Hands-On Exercises](#module-4--practical-hands-on-exercises)
5. [Module 5 – Common Compilation & Runtime Errors](#module-5--common-compilation--runtime-errors)
6. [Module 6 – Techniques, Patterns & Hacks](#module-6--techniques-patterns--hacks)
7. [Module 7 – Interview Questions & Answers](#module-7--interview-questions--answers)
8. [Module 8 – Solutions Appendix](#module-8--solutions-appendix)

---

<div style="page-break-before: always;"></div>

# Module 1 – Stream Basics

### 1.1 What Is a Stream?
A Stream in Java (introduced in Java 8) is a sequence of elements supporting sequential and parallel aggregate operations. Streams DO NOT store data; they convey elements from a source (collection, array, I/O channel, generator) through a pipeline.

### 1.2 Core Characteristics
- No storage (view over data)
- Functional style (map/filter/reduce)
- Lazy (intermediate operations defer execution)
- Can be infinite (generate()/iterate())
- Support parallelism (`parallel()`)

### 1.3 Comparison: Collections vs Streams
| Aspect | Collection | Stream |
|--------|-----------|--------|
| Purpose | Data structure | Data processing pipeline |
| Mutability | Usually mutable | Immutable (operations produce new streams) |
| Consumption | Can be reused | Single-use; terminal operation consumes |
| Traversal | External iteration (`for-each`) | Internal iteration (managed by Stream API) |
| Parallelism | Manual handling | Built-in via `parallel()` |

### 1.4 Stream Pipeline Anatomy
A pipeline consists of:
1. Source
2. Zero or more Intermediate Ops (return Stream)
3. One Terminal Op (produces result / side-effect)

ASCII Diagram:
```
+------------------+    +-----------------+    +-----------------+    +-------------------+
|   Source (List)  | -> |  filter(x>10)   | -> |   map(x*2)      | -> |  collect(toList())|
+------------------+    +-----------------+    +-----------------+    +-------------------+
          |                       |                     |                      |
          v                       v                     v                      v
      Elements               Lazy step             Lazy step             Terminal result
```

### 1.5 Creating Streams
```java
import java.util.stream.*;
import java.util.*;

List<String> names = Arrays.asList("Ana", "Bob", "Chris");
Stream<String> s1 = names.stream();            // From collection
Stream<String> s2 = Stream.of("X", "Y");      // Of values
Stream<Integer> s3 = IntStream.range(0, 5).boxed(); // Primitive stream boxed
Stream<Double> s4 = Stream.generate(Math::random);  // Infinite stream
Stream<Integer> s5 = Stream.iterate(1, n -> n + 2).limit(5); // 1,3,5,7,9
```

### 1.6 Intermediate Operations
| Operation | Type | Stateless/Stateful | Purpose |
|-----------|------|--------------------|---------|
| `map` | Transform | Stateless | Apply function per element |
| `filter` | Select | Stateless | Remove elements not matching predicate |
| `flatMap` | Transform/Flatten | Stateless | Flatten nested structures |
| `distinct` | Refinement | Stateful | Remove duplicates (needs seen-set) |
| `sorted` | Ordering | Stateful | Requires buffering all elements |
| `peek` | Diagnostic | Stateless | Side-effect for debugging |
| `limit`/`skip` | Slicing | Stateful (short-circuiting) | Truncate or skip elements |

### 1.7 Terminal Operations
| Operation | Result Type | Description |
|-----------|-------------|-------------|
| `collect` | Collector result | Mutable reduction into container |
| `reduce` | Optional<T>/T | Immutable associative fold |
| `forEach` | void | Apply side-effect per element |
| `toArray` | Object[] | Materialize array |
| `count` | long | Number of elements |
| `anyMatch` / `allMatch` / `noneMatch` | boolean | Short-circuit predicate tests |
| `findFirst` / `findAny` | Optional<T> | Retrieve element |
| `min` / `max` | Optional<T> | Based on comparator |

### 1.8 Laziness & Short-Circuiting
A stream does nothing until a terminal operation triggers traversal. Short-circuit operations (e.g., `anyMatch`, `limit`) may stop early.

```java
List<Integer> nums = List.of(1,2,3,4,5);
nums.stream()
    .peek(n -> System.out.println("A:" + n))
    .filter(n -> n > 2)
    .peek(n -> System.out.println("B:" + n))
    .findFirst();
// Output demonstrates only necessary elements evaluated.
```

### 1.9 Basic Example (End-to-End)
```java
List<String> words = List.of("stream", "pipeline", "java", "map");
List<Integer> lengths = words.stream()
    .filter(w -> w.length() > 3)
    .map(String::length)
    .sorted()
    .toList(); // Java 16+ convenience
System.out.println(lengths); // [4,5,8]
```

### 1.10 Primitive Streams
- `IntStream`, `LongStream`, `DoubleStream` avoid boxing overhead.
- Provide specialized methods: `sum()`, `average()`, `range()`, `rangeClosed()`.

```java
int total = IntStream.rangeClosed(1, 100).sum();
OptionalDouble avg = IntStream.of(10, 20, 30).average();
```

### 1.11 Common Pitfall: Reusing Streams
```java
Stream<String> base = List.of("a", "b").stream();
long count = base.count();
long again = base.count(); // IllegalStateException: stream has already been operated upon
```

### 1.12 Exercise Set (Basics)
1. Create a stream of integers 1–20; filter multiples of 3; collect to list.
2. Given `List<String>` of words, produce a list of unique word lengths sorted descending.
3. Generate first 10 even numbers using `Stream.iterate`.
4. Count vowels across a list of words using streams.

(See solutions in Module 8.)

---

<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Stream Concepts

### 2.1 Evaluation Model & Fusion
Operations are conceptually fused into a single pass where possible. Stateful operations (e.g., `sorted`) may force buffering.

ASCII Data Flow:
```
[Source] -> [filter] -> [map] -> [distinct] -> [sorted] -> [collect]
     \___________________________/\__________________/\________/
          Stateless Stage(s)        Stateful Stage     Terminal
```

### 2.2 Stateless vs Stateful
- Stateless: does not depend on previously seen elements.
- Stateful: needs global knowledge (sorting, distinct, limit requires counting). Stateful may degrade parallel performance.

### 2.3 Short-Circuiting Operations
- Examples: `anyMatch`, `allMatch`, `noneMatch`, `findFirst`, `findAny`, `limit`.
- They enable early termination of the pipeline.

### 2.4 Parallel Streams
Use `parallel()` or `collection.parallelStream()`.

```java
List<Integer> big = IntStream.range(0, 1_000_000).boxed().toList();
long start = System.nanoTime();
long sum1 = big.stream().reduce(0, Integer::sum);
long mid = System.nanoTime();
long sum2 = big.parallelStream().reduce(0, Integer::sum);
long end = System.nanoTime();
System.out.printf("Sequential=%dms Parallel=%dms%n", (mid-start)/1_000_000, (end-mid)/1_000_000);
```
Caveats:
- Not always faster (thread setup, false sharing)
- Preserve encounter order only if needed
- Avoid shared mutable state

### 2.5 Custom Collectors
Collector is a mutable reduction strategy with 4 components: supplier, accumulator, combiner, finisher (optional), characteristics.

```java
Collector<String, StringBuilder, String> joiningWithBrackets = Collector.of(
    StringBuilder::new,
    (sb, s) -> { if (sb.length() > 0) sb.append(", "); sb.append(s); },
    (sb1, sb2) -> { if (sb1.length()>0 && sb2.length()>0) sb1.append(", "); sb1.append(sb2); return sb1; },
    sb -> "[" + sb.toString() + "]"
);

String out = List.of("A","B","C").stream().collect(joiningWithBrackets); // [A, B, C]
```

### 2.6 Collector Utilities
- `Collectors.groupingBy`
- `Collectors.partitioningBy`
- `Collectors.mapping`
- `Collectors.flatMapping` (Java 9+)
- `Collectors.teeing` (Java 12+)

```java
record Product(String category, double price) {}
List<Product> products = List.of(
    new Product("BOOK", 10.0),
    new Product("BOOK", 15.0),
    new Product("TOY", 25.0)
);

Map<String, Double> avgPriceByCategory = products.stream()
    .collect(Collectors.groupingBy(Product::category, Collectors.averagingDouble(Product::price)));
```

### 2.7 Reduction vs Collection
- `reduce` suits associative, immutable accumulation.
- `collect` suits mutable containers / complex aggregation.

Bad Reduction Example (inefficient due to string concatenation):
```java
String concat = List.of("a","b","c").stream()
    .reduce("", (acc, s) -> acc + s); // Creates many intermediate strings
```
Prefer:
```java
String better = List.of("a","b","c").stream().collect(Collectors.joining());
```

### 2.8 Immutable vs Mutable Patterns
Avoid using external mutable state:
```java
// BAD: race conditions in parallel
List<Integer> source = IntStream.range(0, 1000).boxed().toList();
List<Integer> unsafe = new ArrayList<>();
source.parallelStream().forEach(unsafe::add); // Data race
```
Use collectors or thread-safe structures.

### 2.9 Ordering & findAny
`findAny()` may return any element in parallel mode; `findFirst()` respects encounter order but may restrict performance.

### 2.10 Infinite Streams & Limiting
```java
Stream<String> ids = Stream.generate(() -> UUID.randomUUID().toString()).limit(3);
ids.forEach(System.out::println);
```

### 2.11 Debugging `peek`
Use `peek` only diagnostically:
```java
List<String> result = Stream.of("a","b","error","c")
    .peek(s -> System.out.println("IN:" + s))
    .filter(s -> !s.equals("error"))
    .peek(s -> System.out.println("OUT:" + s))
    .toList();
```

### 2.12 Performance Considerations
- Prefer primitive streams for numeric heavy loops.
- Avoid `sorted()` unless necessary.
- Combine operations to minimize passes (e.g., use `Collectors.summarizingInt`).
- Use `teeing` to derive multiple aggregates in single traversal.

### 2.13 Advanced Example: Multi-level Grouping
```java
record Employee(String dept, String city, int salary) {}
List<Employee> emps = List.of(
    new Employee("ENG","NY",120_000),
    new Employee("ENG","SF",150_000),
    new Employee("HR","NY",80_000),
    new Employee("HR","SF",90_000)
);

Map<String, Map<String, IntSummaryStatistics>> stats = emps.stream()
    .collect(Collectors.groupingBy(Employee::dept,
        Collectors.groupingBy(Employee::city,
            Collectors.summarizingInt(Employee::salary))));
```

### 2.14 Exercise Set (In-Depth)
1. Use a custom collector to build a CSV line from a list of strings.
2. Group a list of words by length and then by first character.
3. Compute both min and max salary using a single stream pass (`teeing`).
4. Create a sliding window of size 3 over an `IntStream` (hint: maintain buffer in custom collector).

(See solutions in Module 8.)

---

<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

Format: Table with Question, Options, Answer & Explanation.

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | What does an intermediate operation return? | A terminal result | A new Stream | A collection | void | B | Intermediate operations return a new Stream enabling chaining. |
| 2 | Which operation is short-circuiting? | map | filter | anyMatch | collect | C | anyMatch may finish early once predicate true. |
| 3 | Which is stateful? | map | distinct | filter | peek | B | distinct needs to remember seen elements. |
| 4 | Best method to concatenate strings efficiently? | reduce with + | collect joining | forEach append external | map then count | B | Collectors.joining uses internal StringBuilder efficiently. |
| 5 | Reusing a consumed stream causes? | Compilation error | Infinite loop | IllegalStateException | Returns empty stream | C | Terminal operation marks stream as consumed. |
| 6 | Parallel streams are always faster? | True | False | Depends on list size only | Depends only on CPU cores | B | Overhead may outweigh benefits for small tasks or poorly partitionable data. |
| 7 | `findAny()` on sequential ordered stream equals? | always first | never first | sometimes first | random each run | C | May return first in sequential; not guaranteed in parallel. |
| 8 | Which Collector gives summary stats? | groupingBy | summarizingInt | counting | mapping | B | summarizingInt returns IntSummaryStatistics. |
| 9 | Primitive stream avoiding boxing? | Stream<Integer> | IntStream | Stream<Object> | LongStream boxed | B | IntStream handles ints without boxing. |
| 10 | Which is invalid? | stream.filter(...).map(...).count() | stream.map(...).filter(...).collect(...) | stream.peek(...).forEach(...) | stream.sorted().findFirst().map(...) | D | After terminal `findFirst` Optional returned; cannot call map on Stream pipeline portion. |
| 11 | `limit(5)` classification? | Terminal | Intermediate short-circuit | Stateless | Collector | B | limit is an intermediate short-circuiting stateful op. |
| 12 | Source for infinite stream? | Arrays.asList() | Stream.generate() | List.stream() | Files.lines() | B | generate produces infinite unless limited. |
| 13 | `reduce(0, Integer::sum)` identity must be? | Non-associative | Neutral element | Largest value | Optional | B | Identity must be neutral for the operation (0 for sum). |
| 14 | Side-effect debug operation? | map | filter | peek | reduce | C | peek for diagnostics. |
| 15 | Terminal operation returning Optional? | collect | reduce(identity,acc) | findFirst | forEach | C | findFirst returns Optional<T>. |

---

### MCQ Exercise: Reflect & Extend
Write two more MCQs about parallel streams and custom collectors. (See solutions in Module 8.)

---

<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

Each exercise defines goal, constraints, hints.

### Exercise 4.1: Word Frequency
Input: List<String> words. Output: Map<String, Long> of counts sorted by descending frequency.
```java
Map<String, Long> freq = words.stream()
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()))
    .entrySet().stream()
    .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue,
        (a,b)->a, LinkedHashMap::new));
```

### Exercise 4.2: Top N Elements (Efficient)
Implement retrieving top 5 salaries without sorting full list.
Hint: Use a bounded priority queue inside a collector.

### Exercise 4.3: Parallel Summarization
Compute total length of all lines in a large file using `Files.lines(Path)` and parallel stream safely.

### Exercise 4.4: Transform & Flatten
Given `List<List<Integer>>`, produce flattened distinct sorted list of squares.
```java
List<Integer> out = listOfLists.stream()
    .flatMap(List::stream)
    .map(n -> n*n)
    .distinct()
    .sorted()
    .toList();
```

### Exercise 4.5: Sliding Window Average
Given IntStream of sensor readings, compute moving average of window size 4.

### Exercise 4.6: Multi-Collector Teeing
Record both sum and average of numbers in one pass.
```java
record Stats(long sum, double avg) {}
Stats stats = IntStream.range(1, 11).boxed()
    .collect(Collectors.teeing(
        Collectors.summingInt(Integer::intValue),
        Collectors.averagingInt(Integer::intValue),
        Stats::new));
```

### Exercise 4.7: Classifying URLs
Group URLs by protocol then domain.

### Exercise 4.8: Error Recovery Pattern
Process lines ignoring malformed integers, collecting valid ones, counting errors.

---

<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Reusing Stream
```java
Stream<String> s = List.of("a","b").stream();
long c1 = s.count();
long c2 = s.count(); // Runtime: IllegalStateException
```
Fix: Recreate stream or store source.

### 5.2 Null Source
```java
List<String> list = null;
list.stream(); // NullPointerException
```
Fix: `Optional.ofNullable(list).orElse(List.of()).stream()`.

### 5.3 Concurrent Modification
```java
List<Integer> nums = new ArrayList<>(List.of(1,2,3));
nums.stream().forEach(n -> { if (n == 2) nums.add(4); }); // ConcurrentModificationException (maybe)
```
Fix: Avoid structural mutation during traversal.

### 5.4 Unintended Boxing
```java
long sum = List.of(1,2,3,4,5).stream().reduce(0, Integer::sum); // Autoboxing overhead
```
Fix: `int sum = IntStream.of(1,2,3,4,5).sum();`

### 5.5 Non-Associative Reduce Combiner Error (Parallel)
```java
int wrong = Arrays.asList(1,2,3,4).parallelStream()
    .reduce(0, (a,b) -> a - b); // Non-associative leads to unpredictable result
```
Fix: Use associative operation.

### 5.6 Shared Mutable State
```java
List<Integer> src = IntStream.range(0, 1000).boxed().toList();
List<Integer> unsafe = new ArrayList<>();
src.parallelStream().forEach(unsafe::add); // Data race
```
Fix: Use `Collectors.toList()` or thread-safe structures.

### 5.7 Order Loss
Using `findAny()` expecting first element in parallel leads to logic bug.

### 5.8 `map` vs `flatMap` Mistake
```java
List<List<Integer>> nested = List.of(List.of(1,2), List.of(3));
Stream<List<Integer>> wrong = nested.stream().map(List::stream); // Stream<Stream<Integer>>
```
Fix: `nested.stream().flatMap(List::stream);`

### 5.9 Overusing `peek` Causing Side Effects
```java
list.stream().peek(list::add).count(); // May cause ConcurrentModificationException
```

### 5.10 Exercise: Identify Errors
Find 3 problems in the following:
```java
Stream<Integer> st = Stream.of(1,2,3);
st.peek(System.out::println); // 1
st.filter(n -> n>1).count();   // 2
long again = st.count();       // 3
```
(Answers in Module 8.)

---

<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Custom Top-N Collector
```java
public static <T> Collector<T, PriorityQueue<T>, List<T>> topN(Comparator<T> cmp, int n) {
    return Collector.of(
        () -> new PriorityQueue<>(n, cmp),
        (pq, t) -> { if (pq.size() < n) pq.offer(t); else if (cmp.compare(t, pq.peek()) > 0) { pq.poll(); pq.offer(t); } },
        (pq1, pq2) -> { pq2.forEach(t -> { if (pq1.size() < n) pq1.offer(t); else if (cmp.compare(t, pq1.peek()) > 0) { pq1.poll(); pq1.offer(t);} }); return pq1; },
        pq -> {
            List<T> out = new ArrayList<>(pq); out.sort(cmp.reversed()); return out; },
        Collector.Characteristics.UNORDERED
    );
}
```

### 6.2 `teeing` Collector Patterns
Combine sum and count to produce average manually (if pre-Java 12 fallback: custom collector).

### 6.3 Windowing Strategy
Use custom collector or `IntStream.range` index pairing.

### 6.4 Stream-Based Caching (Memoization)
For expensive mapping:
```java
Map<String,Integer> cache = new ConcurrentHashMap<>();
List<Integer> lengths = words.stream()
    .map(w -> cache.computeIfAbsent(w, String::length))
    .toList();
```

### 6.5 Partition & Post-Process
```java
Map<Boolean, List<Integer>> parts = IntStream.rangeClosed(1,10).boxed()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

### 6.6 Flatten Optional
```java
Stream<Optional<String>> optStream = Stream.of(Optional.of("A"), Optional.empty());
List<String> present = optStream.flatMap(o -> o.stream()).toList(); // Java 9+ Optional::stream
```

### 6.7 Safe Resource Handling
Use try-with-resources for IO streams:
```java
try (Stream<String> lines = Files.lines(Path.of("data.txt"))) {
    long nonEmpty = lines.filter(l -> !l.isBlank()).count();
}
```

### 6.8 Performance Benchmark Harness Idea
Use JMH for rigorous benchmarking beyond naive `System.nanoTime()`.

### 6.9 Lazy Logging Pattern
```java
logger.debug(() -> list.stream().map(Object::toString).collect(Collectors.joining(",")));
```

### 6.10 Exercise: Write a Collector that Computes Median
Hint: Buffer elements, sort at finish (stateful). Consider trade-offs.

---

<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Difference between `map` and `flatMap`?  
Answer: `map` transforms each element independently; `flatMap` both transforms and flattens nested streams (or collections) into a single stream.  
2. Why are streams single-use?  
Answer: Architectural simplification: after a terminal operation the pipeline resources may be released; reuse risks inconsistent state.  
3. When to prefer `reduce` vs `collect`?  
Answer: Use `reduce` for immutable associative operations (sum, product); use `collect` for mutable containers or multi-field aggregation.  
4. Pitfalls of parallel streams?  
Answer: Overhead, poor data partitioning, false sharing, non-thread-safe side effects, unpredictable ordering with `findAny`.  
5. Explain encounter order.  
Answer: Order derived from source (e.g., `List` preserves insertion order; `HashSet` does not). Some operations respect it; parallel stream may relax with `findAny`.  
6. How does `Collectors.groupingBy` work internally?  
Answer: Uses a mutable Map accumulator: supplier creates map; accumulator inserts grouped entries; combiner merges maps in parallel; finisher possibly identity.  
7. What is a short-circuiting operation?  
Answer: An operation able to complete without processing all elements (e.g., `limit`, `findFirst`, `anyMatch`).  
8. Why is `distinct` potentially expensive?  
Answer: Requires maintaining a set of seen elements; increases memory and prevents streamed pipelining in parallel effectively.  
9. Is `sorted().parallel()` always slower?  
Answer: Not always; large datasets with CPU cores can benefit; but sorting is stateful requiring global compare operations.  
10. Provide example of custom collector components.  
Answer: Supplier (container creation), accumulator (add element), combiner (merge partial), finisher (transform container), characteristics (e.g., UNORDERED).

### Exercise: Draft an answer explaining difference between terminal and intermediate ops with examples.
(See Module 8.)

---

<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. Multiples of 3:
```java
List<Integer> out = IntStream.rangeClosed(1,20).filter(n -> n % 3 == 0).boxed().toList();
```
2. Unique word lengths descending:
```java
List<Integer> lens = words.stream().map(String::length).distinct()
    .sorted(Comparator.reverseOrder()).toList();
```
3. First 10 evens:
```java
List<Integer> evens = Stream.iterate(0, n -> n + 2).limit(10).toList();
```
4. Count vowels:
```java
long vowels = words.stream()
    .flatMapToInt(String::chars)
    .map(Character::toLowerCase)
    .filter(ch -> "aeiou".indexOf(ch) >= 0)
    .count();
```

## Solutions: Module 2 Exercises
1. CSV collector:
```java
Collector<String, StringBuilder, String> csv = Collector.of(
    StringBuilder::new,
    (sb,s) -> { if (sb.length()>0) sb.append(','); sb.append(s); },
    (a,b) -> { if (a.length()>0 && b.length()>0) a.append(','); a.append(b); return a; },
    StringBuilder::toString
);
String line = list.stream().collect(csv);
```
2. Group by length then first char:
```java
Map<Integer, Map<Character, List<String>>> grouped = words.stream()
    .collect(Collectors.groupingBy(String::length,
        Collectors.groupingBy(w -> w.charAt(0))));
```
3. Min & Max with teeing:
```java
record MinMax(int min, int max) {}
MinMax mm = IntStream.of(5,2,9,1).boxed()
    .collect(Collectors.teeing(
        Collectors.minBy(Comparator.naturalOrder()),
        Collectors.maxBy(Comparator.naturalOrder()),
        (minOpt, maxOpt) -> new MinMax(minOpt.orElseThrow(), maxOpt.orElseThrow())
    ));
```
4. Sliding window of size 3 (simple approach):
```java
List<Integer> readings = IntStream.rangeClosed(1,10).boxed().toList();
List<Double> movingAvg = IntStream.range(0, readings.size()-2)
    .mapToDouble(i -> (readings.get(i)+readings.get(i+1)+readings.get(i+2))/3.0)
    .boxed().toList();
```

## Solutions: Module 3 MCQ Extension
Additional MCQs:
1. Parallel stream splits work using? Answer: ForkJoinPool (common). Explanation: Uses ForkJoinPool.commonPool with work-stealing.
2. Custom collector combiner used when? Answer: In parallel execution. Explanation: Sequential pipelines may never invoke combiner; must remain correct.

## Solutions: Module 4 Selected Exercises
Exercise 4.2 (Top 5 salaries):
```java
List<Integer> salaries = List.of(10,50,30,70,90,20,40);
List<Integer> top5 = salaries.stream()
    .collect(topN(Comparator.naturalOrder(), 5));
```
Exercise 4.3 (Parallel length sum):
```java
long total;
try (Stream<String> lines = Files.lines(Path.of("large.txt"))) {
    total = lines.parallel().mapToLong(String::length).sum();
}
```
Exercise 4.5 (Moving average size 4):
```java
List<Integer> data = IntStream.rangeClosed(1,9).boxed().toList();
List<Double> avg4 = IntStream.range(0, data.size()-3)
    .mapToDouble(i -> (data.get(i)+data.get(i+1)+data.get(i+2)+data.get(i+3))/4.0)
    .boxed().toList();
```
Exercise 4.7 (URL grouping):
```java
record Url(String protocol, String domain, String path) {}
List<Url> urls = List.of(
    new Url("https","example.com","/a"),
    new Url("http","example.com","/b"),
    new Url("https","other.io","/x")
);
Map<String, Map<String, List<Url>>> grouped = urls.stream()
    .collect(Collectors.groupingBy(Url::protocol,
        Collectors.groupingBy(Url::domain)));
```
Exercise 4.8 (Error recovery):
```java
List<String> lines = List.of("10","x","25");
class Acc { List<Integer> nums = new ArrayList<>(); int errors = 0; }
Acc acc = lines.stream().collect(Collector.of(
    Acc::new,
    (a,s) -> { try { a.nums.add(Integer.parseInt(s)); } catch (NumberFormatException e) { a.errors++; } },
    (a,b) -> { a.nums.addAll(b.nums); a.errors += b.errors; return a; }
));
System.out.println(acc.nums + " errors=" + acc.errors);
```

## Solutions: Module 5 Error Identification (5.10)
Problems:
1. Missing terminal operation on `peek` line (does nothing).
2. Reuse of same stream variable after partial operation chain mixing.
3. Second `count()` after stream consumed -> IllegalStateException.

## Solutions: Module 6 Median Collector (Sketch)
```java
public static Collector<Integer, List<Integer>, Double> medianCollector() {
    return Collector.of(
        ArrayList::new,
        List::add,
        (a,b) -> { a.addAll(b); return a; },
        list -> { if (list.isEmpty()) return Double.NaN; list.sort(Integer::compareTo); int mid = list.size()/2; return list.size()%2==0 ? (list.get(mid-1)+list.get(mid))/2.0 : list.get(mid)*1.0; }
    );
}
```

## Solutions: Module 7 Exercise (Terminal vs Intermediate)
Terminal operations trigger processing and produce a non-stream result (e.g., `collect`, `count`). Intermediate operations return a stream enabling further composition and are lazy (e.g., `map`, `filter`). A pipeline like `list.stream().filter(...).map(...).count()` only runs filters/maps when `count()` executes.

---

### End Notes
Practice incrementally; favor readability over extreme chaining. Measure performance instead of assuming parallel gains.
