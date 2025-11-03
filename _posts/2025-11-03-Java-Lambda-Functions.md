
# Java Lambda Functions — Comprehensive Training

**Author:** Senior Technical Java Trainer / Staff Software Engineer  
**Audience:** Intermediate–Advanced Java Developers  
**Prerequisites:** Java 8+, Collections, Generics, Basic OOP

---

## Table of Contents
- [Module 1: Basics of Java 8 Lambda Functions](#module-1-basics-of-java-8-lambda-functions)
- [Module 2: Lambda Expressions In-Depth](#module-2-lambda-expressions-in-depth)
- [Module 3: Multiple Choice Questions (MCQs)](#module-3-multiple-choice-questions-mcqs)
- [Module 4: Practical Learning — Hands-on Labs](#module-4-practical-learning--hands-on-labs)
- [Module 5: Techniques, Patterns & Hacks](#module-5-techniques-patterns--hacks)
- [Module 6: Interview Questions & Answers](#module-6-interview-questions--answers)

---


<!-- PAGEBREAK -->


## Module 1: Basics of Java 8 Lambda Functions

### What is a Lambda Expression?
A **lambda expression** is a concise way to represent an **anonymous function**—a block of code that you can pass around and execute later.
Lambdas enable **functional programming** style in Java while interoperating cleanly with the object-oriented model.

### Why Lambdas (Motivation)
- Reduce boilerplate (replace many anonymous inner classes)
- Unlock **Streams API** for declarative data processing
- Improve readability and expressiveness
- Better parallelization potential with streams

### Syntax Overview
```java
(parameters) -> expression
(parameters) -> { statements; }

// Examples
() -> 42
x -> x * x
(a, b) -> a + b
(String s) -> { System.out.println(s); }
```

### Functional Interfaces
An interface with exactly **one abstract method** (SAM). Use `@FunctionalInterface` to signal intent.
Common built-ins (in `java.util.function`):
- `Predicate<T>`: boolean test(T t)
- `Function<T,R>`: R apply(T t)
- `Consumer<T>`: void accept(T t)
- `Supplier<T>`: T get()
- `UnaryOperator<T>` and `BinaryOperator<T>`
- `BiPredicate<T,U>`, `BiFunction<T,U,R>`, `BiConsumer<T,U>`

**Example:**
```java
Predicate<Integer> isEven = n -> n % 2 == 0;
Function<String, Integer> length = s -> s.length();
Consumer<String> printer = msg -> System.out.println(msg);
Supplier<Double> random = () -> Math.random();
```

### Lambdas vs Anonymous Inner Classes
- **`this`** in a lambda refers to the **enclosing instance**; in anonymous class, it refers to the anonymous class itself.
- Lambdas **capture effectively-final** variables from surrounding scope; same rule as anonymous classes but without generating a new class.
- Lambdas are implemented via `invokedynamic` (not a synthetic class file), generally more lightweight.

### ASCII Diagram: Where Lambdas Fit
```
+-------------------+     provides SAM      +------------------------+
| Functional        |  <------------------  | Lambda Expression      |
| Interface (SAM)   |   implements SAM      |  (code block)          |
+-------------------+                       +------------------------+
            |                                              |
            v                                              v
     +--------------+                              +----------------+
     | Method Ref   |   interchangeable with       | Streams API    |
     | Class::meth  |   lambdas (when signatures)  | map/filter/... |
     +--------------+                              +----------------+
```

### Quick Start Examples
```java
// Runnable with Lambda
new Thread(() -> System.out.println("Hello Lambda!")).start();

// Comparator with Lambda
List<String> names = Arrays.asList("Suresh", "Anita", "Pratik", "Dayou");
names.sort((a, b) -> Integer.compare(a.length(), b.length()));

// Method Reference variant
names.sort(Comparator.comparingInt(String::length));
```


<!-- PAGEBREAK -->


## Module 2: Lambda Expressions In-Depth

### Type Inference & Target Typing
The compiler infers parameter and return types from the **target functional interface**.
```java
List<Integer> nums = List.of(1,2,3,4,5);
nums.stream().map(n -> n * n); // n is inferred as Integer
```

### Variable Capture & Effectively Final
Variables captured from surrounding scope must be **effectively final**.
```java
int base = 10; // effectively final
Function<Integer, Integer> addBase = x -> x + base; // OK

// base++; // would break "effectively final" rule
```

When you write a lambda (or an anonymous class), it can **use variables from its enclosing scope**. This is called **capture**. Example:
```java
int base = 10; // effectively final
Function<Integer, Integer> addBase = x -> x + base; // captures 'base'
System.out.println(addBase.apply(5)); // 15
```

**How does capture work conceptually?**

For local variables (including method parameters) used inside a lambda:

- The lambda captures the value of those variables at the moment the lambda is created. This is often described as “capture by value” (for primitives) and “capture the reference by value” (for reference types).
- For fields (instance or static), lambdas don’t need them to be effectively final; they are accessed normally via the object or class and can be mutated.

**Why “effectively final”?**
Java requires that captured local variables (from the enclosing method) be effectively final. This means:

You don’t have to write the final keyword.
But the variable’s value must not be changed after it’s assigned.
```java
int base = 10;                   // effectively final
Function<Integer, Integer> f = x -> x + base; // OK

// base++;                       // ❌ breaks effectively final rule
```

**Effective final 'Not allowed'**
```java
List<String> list = new ArrayList<>(); // reference is fixed
Consumer<String> add = s -> list.add(s); // OK: we mutate the list, not the reference

// list = new ArrayList<>();           // ❌ would break effectively final (reassignment)
```

“Effectively final” is about the variable’s reference, not about the object’s internal state.
```java
List<String> list = new ArrayList<>(); // reference is fixed
Consumer<String> add = s -> list.add(s); // OK: we mutate the list, not the reference

// list = new ArrayList<>();           // ❌ would break effectively final (reassignment)
```

Using counters inside lambdas is not **'Effective final'**

```java
int count = 0;
List<String> names = List.of("a","b","c");

// ❌ compile error: count is not effectively final
names.forEach(n -> { /* count++ */ });
```

Workaround
```java
AtomicInteger counter = new AtomicInteger(0);
names.forEach(n -> counter.incrementAndGet());
System.out.println(counter.get());
```

Try/catch and effectively final

```java
String value;
try {
    value = "A";
} catch (Exception e) {
    value = "B";    // two assignments → not effectively final
}
Supplier<String> s = () -> value; // ❌ compile error
```

Workaround
```java
String value;
try {
    value = compute();
} catch (Exception e) {
    value = fallback();
}
final String finalValue = value;  // now final variable
Supplier<String> s = () -> finalValue; // OK
```

Effective final 'Reassignment in loops'
```java
List<Runnable> tasks = new ArrayList<>();
for (int i = 0; i < 3; i++) {
    // If you refer to 'i' directly in some older anonymous-class idioms, you risk capturing the changing variable.
    // With lambdas, 'i' must be effectively final per iteration.
    int snapshot = i; // create a new effectively-final variable per iteration
    tasks.add(() -> System.out.println(snapshot));
}
tasks.forEach(Runnable::run); // prints 0, 1, 2 (as expected)
```

**ASCII Diagram: Capture**
```
[Enclosing Scope]
  base = 10 (read-only for lambda)
       |
       v
lambda x -> x + base
```

### Method References
- Static: `ClassName::staticMethod`
- Instance (bound): `instance::instanceMethod`
- Instance (unbound): `ClassName::instanceMethod`
- Constructor: `ClassName::new`

```java
Function<String, Integer> parse = Integer::parseInt;     // static
Supplier<List<String>> mkList = ArrayList::new;          // ctor
Comparator<String> byLen = Comparator.comparingInt(String::length); // unbound instance
```

### Streams + Lambdas Essentials
```java
List<String> users = List.of("alice","bob","carol","dave");
List<String> result = users.stream()
    .filter(u -> u.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .toList();
```

**ASCII Flow (map-filter-reduce)**
```
[source] -> [map] -> [filter] -> [sorted] -> [collect/toList]
```

### Compose Predicates / Functions
```java
Predicate<String> nonEmpty = s -> !s.isEmpty();
Predicate<String> longer5 = s -> s.length() > 5;
Predicate<String> good = nonEmpty.and(longer5);

Function<String, Integer> length = String::length;
Function<Integer, Integer> square = n -> n*n;
Function<String, Integer> lengthSquared = length.andThen(square);
```

### Exceptions in Lambdas (Checked)
Lambdas cannot widen checked exceptions. Use wrapping helpers.
```java
@FunctionalInterface
interface ThrowingFunction<T,R,E extends Exception> {
    R apply(T t) throws E;
}

static <T,R> Function<T,R> unchecked(ThrowingFunction<T,R,?> f) {
    return t -> {
        try { return f.apply(t); }
        catch (Exception e) { throw new RuntimeException(e); }
    };
}

// Usage
Files.lines(Path.of("data.txt"))
     .map(unchecked(line -> line.toUpperCase()))
     .forEach(System.out::println);
```

### Performance Considerations
- Avoid creating lambdas in tight loops if captured state changes; consider caching.
- Prefer **primitive streams** (`IntStream`, etc.) to avoid boxing.
- Use **parallel streams** only with large, CPU-bound, stateless operations and non-contentious collectors.

**ASCII: Parallel Streams Caveat**
```
+------------+    +------------+    +------------+
| split work| => |  threads   | => | combine    |
+------------+    +------------+    +------------+
     \             Beware: shared state, ordering, IO
      \            can hurt performance or correctness
           ```

### `this` and Scope Differences
In a lambda, `this` refers to the **enclosing class instance**; anonymous classes have their own `this`.
```java
class Demo {
    void run() {
        Runnable r = () -> System.out.println(this.toString()); // enclosing
        r.run();
    }
}
```


<!-- PAGEBREAK -->


## Module 3: Multiple Choice Questions (MCQs)

**Instructions:** Select the best answer. Answers and explanations provided.

| # | Question | Options | Answer | Explanation |
|---|----------|---------|--------|-------------|
| 1 | Lambdas can be assigned to which types? | a) Any interface  b) Only abstract classes  c) Functional interfaces  d) Enums | c | Lambdas target **functional interfaces** (single abstract method). |
| 2 | What does "effectively final" mean? | a) Must be `final` keyword  b) Value not changed after init  c) Can change any time  d) Only for fields | b | Local captured vars must not change after assignment. |
| 3 | `this` inside a lambda refers to? | a) The lambda  b) Anonymous class  c) Enclosing instance  d) Static context | c | Lambdas do not introduce a new scope for `this`. |
| 4 | Which is NOT a built-in functional interface? | a) Predicate  b) Projector  c) Supplier  d) Consumer | b | `Projector` doesn’t exist in `java.util.function`. |
| 5 | Valid method reference? | a) `String::new`  b) `List::size`  c) `Math::max`  d) All of the above | d | All map to SAM signatures in the right context. |
| 6 | Streams are | a) Stateful collections  b) Reusable sequences  c) Single-use pipelines  d) Arrays | c | Streams are consumed once. |
| 7 | Parallel streams are best for | a) IO-bound ops  b) Small lists  c) CPU-bound, large datasets  d) UI tasks | c | Parallelism shines for large, CPU-bound tasks. |
| 8 | Which compiles? | a) `x -> { return; }`  b) `() -> { return 1; }`  c) `(a,b) -> a + b`  d) All | d | All match suitable targets. |
| 9 | `Comparator.comparingInt(String::length)` returns | a) Comparator<String> | a | Specialized comparator by int key. |
| 10 | Which throws at runtime if mishandled? | a) Checked inside lambda  b) Swallowed exceptions  c) Unchecked wrapped  d) All of the above | d | Need explicit handling; wrapping often used. |
| 11 | Which collects to immutable list (Java 16+)? | a) `collect(Collectors.toList())`  b) `toList()` terminal op  c) `new ArrayList<>(...)`  d) `Stream.of(...).toList()` | b | `toList()` returns unmodifiable list. |
| 12 | Unbound instance method ref `String::compareToIgnoreCase` has target SAM of | a) BiFunction<String,String,Integer>  b) Comparator<String>  c) IntBinaryOperator  d) Runnable | b | Matches `(a,b) -> a.compareToIgnoreCase(b)`. |
| 13 | Capturing mutable external state in parallel stream | a) Safe  b) Unsafe  c) Faster  d) Required | b | Leads to race conditions. |
| 14 | `Predicate<T>` default methods | a) `and`, `or`, `negate`  b) `map`, `flatMap`  c) `apply`  d) `accept` | a | Composition helpers exist. |
| 15 | Constructor reference example | a) `HashMap::new`  b) `Map::put`  c) `Collections::emptyList`  d) `System::gc` | a | Constructor ref for SAM like `Supplier<Map>`. |
| 16 | Lambdas can declare type of parameters | a) Never  b) Optional  c) Always required  d) Only with `var` | b | Types can be inferred or explicit, Java 11+ allows `var`. |
| 17 | `Stream.peek` is for | a) Production side effects  b) Debugging/inspection  c) Sorting  d) Error handling | b | Use primarily for debugging; no guaranteed ordering of side effects. |
| 18 | Which is true about `map` vs `flatMap` | a) Both same  b) `flatMap` flattens nested streams  c) `map` throws  d) `flatMap` only for Optionals | b | `flatMap` merges nested structures. |
| 19 | Lambda serialization | a) Guaranteed stable form  b) Implementation detail  c) Always supported  d) Required | b | Lambda serialization is not stable API; avoid relying on it. |
| 20 | `Collectors.groupingBy` with parallel stream needs | a) Concurrent collector  b) Synchronized list  c) No change  d) ForkJoin lock | a | Prefer `groupingByConcurrent` when appropriate.



<!-- PAGEBREAK -->


## Module 4: Practical Learning — Hands-on Labs

### Lab 1: Sort by Multiple Keys
**Task:** Sort employees by department, then by descending salary, then by name.

```java
record Emp(String name, String dept, int salary) {}

List<Emp> emps = List.of(
    new Emp("Suresh","Eng", 210),
    new Emp("Anita","Eng", 180),
    new Emp("Pratik","PM", 220),
    new Emp("Dayou","Eng", 210)
);

List<Emp> sorted = emps.stream()
    .sorted(
        Comparator.comparing(Emp::dept)
                  .thenComparing(Comparator.comparingInt(Emp::salary).reversed())
                  .thenComparing(Emp::name)
    )
    .toList();
```

### Lab 2: Compose Predicates
```java
Predicate<String> nonNull = Objects::nonNull;
Predicate<String> nonBlank = s -> !s.isBlank();
Predicate<String> longEnough = s -> s.length() >= 5;

Predicate<String> good = nonNull.and(nonBlank).and(longEnough);

List<String> inputs = List.of("", "  ", "hello", null, "lambda");
List<String> cleaned = inputs.stream().filter(good).toList();
```

### Lab 3: Map-Filter-Reduce
```java
int sumSquares = IntStream.rangeClosed(1, 100)
    .map(n -> n * n)
    .filter(n -> n % 2 == 0)
    .sum();
```

### Lab 4: Checked Exceptions Wrapper
```java
@FunctionalInterface
interface ThrowingConsumer<T, E extends Exception> { void accept(T t) throws E; }

static <T> Consumer<T> unchecked(ThrowingConsumer<T, ?> c) {
    return t -> { try { c.accept(t); } catch (Exception e) { throw new RuntimeException(e); } };
}

Files.lines(Path.of("input.txt")).forEach(unchecked(line -> {
    if (line.contains("ERR")) throw new IOException("Bad line");
    System.out.println(line);
}));
```

### Lab 5: Grouping & Aggregations
```java
record Txn(String user, String cat, double amount) {}

Map<String, Double> totals = List.of(
    new Txn("alice","food",12.5), new Txn("bob","tech",200),
    new Txn("alice","tech",99.0), new Txn("bob","food",13.0)
).stream().collect(
    Collectors.groupingBy(Txn::user, Collectors.summingDouble(Txn::amount))
);
```

### ASCII: Stream Pipeline Anatomy
```
+--------+    +--------+    +----------+    +---------+    +----------+
| source | -> | map    | -> | filter   | -> | sort    | -> | collect  |
+--------+    +--------+    +----------+    +---------+    +----------+
     |             |             |               |              |
   Iterable     Function      Predicate       Comparator      Collector
```

### Stretch: Custom Collector Skeleton
```java
public static <T> Collector<T, StringBuilder, String> joiningWith(String sep) {
    return Collector.of(
        StringBuilder::new,
        (sb, t) -> { if (sb.length() > 0) sb.append(sep); sb.append(t); },
        (a, b) -> { if (a.length()>0 && b.length()>0) a.append(sep); return a.append(b); },
        StringBuilder::toString
    );
}
```


<!-- PAGEBREAK -->


## Module 5: Techniques, Patterns & Hacks

### 1) Comparator Patterns
```java
Comparator<Person> cmp = Comparator
    .comparing(Person::lastName, String.CASE_INSENSITIVE_ORDER)
    .thenComparingInt(Person::age)
    .thenComparing(Person::firstName);
```

### 2) Predicate Algebra
```java
Predicate<Path> isJava = p -> p.toString().endsWith(".java");
Predicate<Path> isLarge = p -> p.toFile().length() > 1024 * 1024;
Predicate<Path> interesting = isJava.and(isLarge).negate();
```

### 3) Currying & Partial Application (with helpers)
```java
static <A,B,R> Function<B,R> curry(BiFunction<A,B,R> f, A a) {
    return b -> f.apply(a, b);
}

BiFunction<Integer,Integer,Integer> pow = (a,b) -> (int)Math.pow(a,b);
Function<Integer,Integer> square = curry(pow, 2);
```

### 4) Lazy Evaluation with Suppliers
```java
void log(Level lvl, Supplier<String> msg) {
    if (logger.isLoggable(lvl)) logger.log(lvl, msg.get());
}

log(Level.FINE, () -> expensiveToBuildMessage());
```

### 5) Memoization Sketch
```java
static <T,R> Function<T,R> memoize(Function<T,R> f) {
    Map<T,R> cache = new ConcurrentHashMap<>();
    return t -> cache.computeIfAbsent(t, f);
}
```

### 6) Optional Pipelines with Lambdas
```java
Optional.of("  lambda  ")
    .map(String::trim)
    .filter(s -> !s.isEmpty())
    .ifPresent(System.out::println);
```

### 7) Exception Wrappers (generic)
```java
static <T> Consumer<T> sneaky(ThrowingConsumer<T,Exception> c) {
    return t -> { try { c.accept(t); } catch (Exception e) { throw new RuntimeException(e); } };
}
```

### ASCII: Choosing Stream vs Loop
```
+-------------------+      yes       +------------------------+
| CPU-bound & large |  ----------->  | Consider parallel     |
| dataset, pure fn? |               | stream (measure!)     |
+-------------------+               +------------------------+
          | no
          v
    +-----------+      yes       +-----------------+
    | Simple?   |  ----------->  | Stream pipeline |
    +-----------+               +-----------------+
          | no
          v
       +--------+
       |  Loop  |
       +--------+
```


<!-- PAGEBREAK -->


## Module 6: Interview Questions & Answers

**Q1. What is a functional interface? Why `@FunctionalInterface`?**  
**A:** An interface with a single abstract method enabling lambda targets. Annotation documents intent and lets compiler validate.

**Q2. Difference between lambda and anonymous inner class?**  
**A:** `this` binding (enclosing vs new scope), lighter implementation (`invokedynamic`), cleaner syntax; both capture effectively-final vars.

**Q3. How do you handle checked exceptions in lambdas?**  
**A:** Wrap with helper that converts to unchecked or design APIs to avoid checked exceptions.

**Q4. Explain method references types.**  
**A:** Static, instance (bound/unbound), constructor refs; must match target SAM signature.

**Q5. When to use parallel streams?**  
**A:** Large, CPU-bound, stateless operations; avoid IO-bound and shared mutable state; always measure.

**Q6. What does effectively-final mean?**  
**A:** Variable’s value is not modified after assignment, even if not declared `final`.

**Q7. Stream pipeline characteristics?**  
**A:** Lazy intermediate ops, terminal op triggers execution, single-use, potential short-circuiting.

**Q8. Can lambdas be serialized?**  
**A:** Not reliably; representation is an implementation detail—avoid relying on serialization.

**Q9. `this` and variable shadowing?**  
**A:** Lambdas can’t shadow local variables; `this` refers to enclosing instance.

**Q10. Performance of lambdas vs loops?**  
**A:** Comparable for many cases; streams add overhead for tiny datasets; primitives streams help by avoiding boxing.

