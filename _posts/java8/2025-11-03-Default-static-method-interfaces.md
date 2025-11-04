# Java Interface Default & Static Methods – In-Depth Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

---
## Table of Contents
1. [Module 1 – Basics](#module-1--basics)
2. [Module 2 – In-Depth Concepts](#module-2--in-depth-concepts)
3. [Module 3 – Multiple Choice Questions (MCQs)](#module-3--multiple-choice-questions-mcqs)
4. [Module 4 – Practical Hands-On Exercises](#module-4--practical-hands-on-exercises)
5. [Module 5 – Common Compilation & Runtime Errors](#module-5--common-compilation--runtime-errors)
6. [Module 6 – Techniques, Patterns & Hacks](#module-6--techniques-patterns--hacks)
7. [Module 7 – Interview Questions & Answers](#module-7--interview-questions--answers)
8. [Module 8 – Solutions Appendix](#module-8--solutions-appendix)

---
<div style="page-break-before: always;"></div>

# Module 1 – Basics

### 1.1 Why Default & Static Methods Were Introduced
Java 8 added default & static interface methods to evolve APIs (e.g., `java.util.Collection`) without breaking existing implementations. They allow adding behavior to interfaces while preserving binary compatibility.

### 1.2 Definitions
- Default Method: Interface method with `default` keyword providing a body; inheritable by implementing classes.
- Static Method: Interface-scoped utility method (belongs to interface namespace) invoked via `InterfaceName.method()`.

### 1.3 Syntax Overview
```java
public interface Logger {
    void log(String msg); // abstract method
    default void info(String msg) { log("INFO: " + msg); }
    static Logger stdout() { return System.out::println; }
}
```

### 1.4 ASCII Concept Diagram
```
+---------------------------+
|         Interface         |
|  abstract methods         |  <- must be implemented by class
|  default methods (impl)   |  <- optional override
|  static methods (utility) |  <- called via InterfaceName
+---------------------------+
            ^
            |
     +-------------+
     |  Class Impl |
     | overrides?  |
     +-------------+
```

### 1.5 Invocation Examples
```java
Logger logger = Logger.stdout(); // static method call
logger.info("App started");      // default method using abstract log()
```

### 1.6 When to Use Default Methods
- Provide minor behavioral extensions (e.g., `Iterable.forEach`)
- Backward compatibility for evolving interfaces
- Provide mixin-like behavior (light multiple inheritance of behavior)

### 1.7 When NOT to Use
- Business logic unrelated to core abstraction
- Hidden statefulness (default methods should be side-effect limited)
- Large hierarchies causing diamond conflicts

### 1.8 Simple Example
```java
interface Shape {
    double area();
    default boolean isLarge() { return area() > 100.0; }
}
class Circle implements Shape {
    private final double r;
    Circle(double r) { this.r = r; }
    public double area() { return Math.PI * r * r; }
}
System.out.println(new Circle(10).isLarge()); // true
```

### 1.9 Exercise Set (Basics)
1. Create an interface `Scaler` with abstract `scale(int x)` and default `doubleScale(int x)` returning `scale(x*2)`.
2. Add a static factory to `Scaler` returning lambda implementation.
3. Create a default method performing argument validation then delegating.

(See Module 8 for solutions.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Concepts

### 2.1 Method Resolution Rules
Precedence order:
1. Class methods (concrete or abstract in superclass) override interface defaults.
2. Most specific interface wins if single inheritance path.
3. Diamond conflict requires explicit override.
4. Static methods are NOT inherited (cannot be called via instance reference).

### 2.2 Diamond Problem
When two interfaces provide defaults with same signature, implementing class must disambiguate.
```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }
class C implements A, B { // compile error unless overridden
    @Override public void hello() { A.super.hello(); B.super.hello(); }
}
```

ASCII Conflict Diagram:
```
    +-------+        +-------+
    |   A   |        |   B   |
    |hello()|        |hello()|
    +---+---+        +---+---+
         \             /
          \           /
            \       /
             +-----+
             |  C  |  <- must override hello()
             +-----+
```

### 2.3 Using `InterfaceName.super.method()`
Inside implementing class override you can call specific default:
```java
class Hybrid implements A, B {
    @Override public void hello() { A.super.hello(); }
}
```

### 2.4 Extension vs Utility
- Default methods: "vertical" extension of behavior.
- Static methods: "horizontal" utility or factory.

### 2.5 Static Method Use Cases
- Factories returning implementations.
- Validation utilities tied to interface contract.
- Composition helpers.

### 2.6 Private Methods in Interfaces (Java 9+)
Facilitate refactoring shared code among multiple default/static methods.
```java
interface Parser {
    private static String trim(String s){ return s == null? "" : s.trim(); }
    static String normalize(String s){ return trim(s).toLowerCase(); }
    private String ensure(String s){ return s == null? "" : s; }
    default boolean isBlank(String s){ return ensure(s).isBlank(); }
}
```

### 2.7 Multiple Inheritance of Behavior vs State
Interfaces provide behavior WITHOUT state inheritance (aside from constants), reducing complexity relative to classes.

### 2.8 Evolution of Java Libraries
Example: `Collection` gained `removeIf`, `forEach`, `stream` via default methods in Java 8.

### 2.9 Performance Considerations
Default methods are virtual dispatch similar to other instance methods; static methods resolved at compile-time (no dynamic dispatch). Private interface methods are static or instance depending on declaration.

### 2.10 Overriding Strategy Patterns via Default
```java
interface Retryable {
    default int maxRetries() { return 3; }
    default boolean shouldRetry(Exception e) { return true; }
}
```
Classes override selectively.

### 2.11 Combining Functional Interfaces with Defaults
```java
@FunctionalInterface
interface AdvancedFunction<T,R> extends java.util.function.Function<T,R> {
    default R applyWithLogging(T t) {
        System.out.println("Applying to " + t);
        return apply(t);
    }
}
```

### 2.12 Exercise Set (In-Depth)
1. Add a private helper method to reduce duplication between two defaults.
2. Resolve a diamond conflict calling both parent defaults.
3. Create an interface `Metrics` with static factory bundling multiple functional interfaces.
4. Extend a functional interface adding an additional default convenience method.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Default method keyword? | `default` | `def` | `extend` | none | A | Java uses `default` keyword. |
| 2 | Static method invocation? | instance.static() | Interface.static() | object.static() | import then call | B | Must use interface name. |
| 3 | Are static interface methods inherited? | Yes | No | Only to subinterfaces | Only to classes | B | Not inherited; accessible via interface name. |
| 4 | Diamond conflict resolution? | Automatic | JVM chooses | Must override | Random | C | Must override to disambiguate. |
| 5 | Private interface methods introduced in? | Java 8 | Java 9 | Java 10 | Java 11 | B | Added in Java 9 onward. |
| 6 | Functional interface + default methods? | Illegal | Allowed | Requires abstract class | Deprecated | B | Additional defaults allowed. |
| 7 | `@FunctionalInterface` permits how many abstract methods? | 1 | 2 | Any | 0 | A | Exactly one abstract method. |
| 8 | Static methods can access instance fields? | Yes | No | Only protected | If synchronized | B | No instance context. |
| 9 | Default method override in class uses? | `super.default()` | `Interface.super.method()` | `this.default()` | `base.method()` | B | Use InterfaceName.super.method(). |
| 10 | Reason to add default methods? | Break APIs | Remove polymorphism | Backward compatibility | Replace inheritance | C | Allows evolving interfaces w/out breaking impls. |
| 11 | Private interface method accessibility? | Anywhere | Within same package | Only within interface methods | Through subclass | C | Only callable from other methods in same interface. |
| 12 | Static interface methods vs utility class? | Slower | Namespaced logically | Must be in abstract class | Uncallable | B | Provide namespacing in abstraction. |
| 13 | Multiple defaults with same signature? | Last one wins | Compile error unless overridden | Picks alphabetical | Runtime exception | B | Must override to resolve. |
| 14 | Can default method be synchronized? | Yes | No | Only static | Only volatile | A | They are normal instance methods. |
| 15 | Private instance interface method can call? | abstract methods | unrelated class privates | external private | protected fields | A | Can call abstract methods implemented later. |

### MCQ Extension Exercise
Draft 2 more MCQs about private interface methods and functional interface extension.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Factory Static Method
Create interface `Connector` with static `of(String url)` returning lambda printing connect message.

### Exercise 4.2: Validation Default
Default method `safeSend(String msg)` validates not blank then delegates to abstract `send`.

### Exercise 4.3: Mixin Behavior
Interface `Timestamped` adds `default long now()` and `default void logWithTs(String msg)` requiring abstract `log(String)`.

### Exercise 4.4: Strategy Override
Override default retry count in subclass to 5.

### Exercise 4.5: Diamond Combination
Two interfaces `Left` and `Right` both define `default tag()`. Implementing class returns combined string.

### Exercise 4.6: Private Helper Consolidation
Refactor two defaults sharing trimming logic into private method.

### Exercise 4.7: Functional Extension
Extend `Predicate<T>` adding default `negateTwice()` returning original predicate result.

### Exercise 4.8: Static Composition Helper
Static method combining two `UnaryOperator<Integer>` producing third.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Diamond Conflict
```java
interface A { default void go(){ System.out.println("A"); } }
interface B { default void go(){ System.out.println("B"); } }
class C implements A,B {} // Compile error: duplicate default methods
```

### 5.2 Static Method via Instance (Illegal)
```java
interface Util { static void f(){ } }
class Impl implements Util {}
Impl i = new Impl();
i.f(); // Compile error: static method not inherited
```
Correct: `Util.f();`

### 5.3 Multiple Abstract Methods in Functional Interface
```java
@FunctionalInterface
interface Bad { void a(); void b(); } // Error: not a functional interface
```

### 5.4 Misusing `super` in Interface Context
```java
interface X { default void m(){ System.out.println("X"); } }
class Y implements X {
    public void call(){ super.m(); } // Compile error: super used wrongly outside override
}
```
Correct usage:
```java
class Y implements X {
    @Override public void m(){ X.super.m(); }
    public void call(){ m(); }
}
```

### 5.5 Private Method Accessibility
```java
interface P { private void h(){} }
class Q implements P { void test(){ h(); } } // Compile error: h() not visible
```

### 5.6 Static Method Shadowing Misconception
Static methods are hidden not overridden; dynamic dispatch not applied.

### 5.7 Abstract + Default Clash
```java
interface Mix { default void run(){} }
class Impl implements Mix { public void run(){} } // Valid override; no clash
```
Clarify: permissible.

### 5.8 Exercise: Identify 3 Errors
```java
@FunctionalInterface
interface Multi { void x(); default void y(){} static void z(){} private void w(){} }
class Test implements Multi { void test(){ z(); w(); y(); } }
```
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Interface as Namespaced Factory
```java
interface Parser {
    static Parser json() { return s -> System.out.println("JSON:" + s); }
    static Parser xml()  { return s -> System.out.println("XML:" + s); }
    void parse(String input);
    default void parseAll(List<String> inputs){ inputs.forEach(this::parse); }
}
```

### 6.2 Layered Defaults
Provide simple default, let subclasses override advanced.

### 6.3 Capability Composition
Combine small interfaces with defaults to form capabilities instead of deep class inheritance.

### 6.4 Private Method Refactoring
Use private interface method to centralize validation logic.

### 6.5 Static Builder Pattern
```java
interface Builder<T> {
    T build();
    static <T> Builder<T> of(Supplier<T> sup){ return sup::get; }
}
```

### 6.6 Memoization Utility
```java
interface Memoizer {
    static <T,R> Function<T,R> memo(Function<T,R> f){
        Map<T,R> cache = new ConcurrentHashMap<>();
        return t -> cache.computeIfAbsent(t, f);
    }
}
```

### 6.7 Decorating via Default
```java
interface Service {
    String execute(String in);
    default String timed(String in){ long s=System.nanoTime(); String r=execute(in); long e=System.nanoTime(); return r + " ("+(e-s)+"ns)"; }
}
```

### 6.8 Fallback Defaults
```java
interface Resilient {
    default String fetch(){ return "fallback"; }
}
```

### 6.9 Multiple Behavior Mixins
```java
interface Loggable { default void log(String m){ System.out.println(m); } }
interface Auditable { default void audit(String m){ System.out.println("AUDIT:"+m); } }
class Ops implements Loggable, Auditable {}
```

### 6.10 Exercise: Compose three capability interfaces into one class using all defaults.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Why were default methods necessary in Java 8?  
Backward-compatible interface evolution (e.g., adding `stream()` to `Collection`).
2. Difference between default and abstract method?  
Default has body and optional override; abstract has no body and must be implemented.
3. Can a static interface method access instance data?  
No; no `this` context.
4. How to resolve multiple default method conflict?  
Override in class and explicitly choose which parent using `InterfaceName.super.method()`.
5. Are private interface methods part of public API?  
No; only internal reuse mechanism.
6. What is a functional interface?  
Interface with exactly one abstract method; may include any number of default/static/private methods.
7. Can you mark interface method `protected`?  
No; interface methods are implicitly public (except private helpers).
8. Why static factories in interface over separate utility class?  
Improves discoverability & cohesion tying creation to contract.
9. How does multiple inheritance differ for interfaces vs classes?  
Interfaces allow multiple behavioral inheritance; classes single parent state/behavior chain.
10. When should you avoid default methods?  
Complex stateful logic, large hierarchies, or when extension better handled by abstract base class.

### Exercise: Draft explanation differentiating static vs default responsibilities.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. `Scaler`:
```java
@FunctionalInterface
interface Scaler {
    int scale(int x);
    default int doubleScale(int x){ return scale(x * 2); }
    static Scaler identity(){ return x -> x; }
}
```
2. Static factory returning lambda shown above.
3. Validation default:
```java
default int safeScale(int x){ if(x < 0) throw new IllegalArgumentException(); return scale(x); }
```

## Solutions: Module 2 Exercises
1. Private helper:
```java
interface Cleaner {
    private String trim(String s){ return s==null? "" : s.trim(); }
    default String normalize(String s){ return trim(s).toLowerCase(); }
    default boolean isEmpty(String s){ return trim(s).isEmpty(); }
}
```
2. Diamond resolution:
```java
class Combo implements A,B { @Override public void hello(){ A.super.hello(); B.super.hello(); } }
```
3. `Metrics` factory:
```java
interface Metrics {
    int value();
    static Metrics of(Supplier<Integer> sup){ return sup::get; }
}
```
4. Extended functional interface:
```java
@FunctionalInterface
interface XForm<T,R> extends Function<T,R> { default R logApply(T t){ System.out.println(t); return apply(t); } }
```

## Solutions: Module 3 MCQ Extension
1. Private interface method call location? Answer: Only inside interface methods. Explanation: Scope restricted.
2. Functional interface extra defaults allowed? Answer: Yes. Explanation: Only abstract count matters.

## Solutions: Module 4 Selected Exercises
Exercise 4.1:
```java
interface Connector { void connect(); static Connector of(String url){ return () -> System.out.println("Connecting to " + url); } }
```
Exercise 4.2:
```java
interface Sender { void send(String msg); default void safeSend(String msg){ if(msg==null||msg.isBlank()) throw new IllegalArgumentException(); send(msg); } }
```
Exercise 4.5:
```java
interface Left { default String tag(){ return "L"; } }
interface Right { default String tag(){ return "R"; } }
class Both implements Left, Right { @Override public String tag(){ return Left.super.tag() + Right.super.tag(); } }
```
Exercise 4.7:
```java
interface AdvPredicate<T> extends Predicate<T> { default boolean negateTwice(T t){ return test(t); } }
```
Exercise 4.8:
```java
interface Combiner { static UnaryOperator<Integer> combine(UnaryOperator<Integer> a, UnaryOperator<Integer> b){ return x -> b.apply(a.apply(x)); } }
```

## Solutions: Module 5 Error Identification (5.8)
Errors:
1. Using private method `w()` outside interface (from class) – not visible.
2. Calling static `z()` without interface qualifier inside class method (needs `Multi.z();`).
3. `test()` not public; implementing interface method `x()` must be public if defined (not in snippet but potential oversight).

## Solutions: Module 6 Composition Exercise
```java
interface Loggable { default void log(String m){ System.out.println(m); } }
interface Timestamped { default long now(){ return System.currentTimeMillis(); } }
interface Reportable { default void report(String m){ System.out.println("REPORT:" + m); } }
class Ops implements Loggable, Timestamped, Reportable { }
```

## Solutions: Module 7 Exercise
Static vs Default: Static methods belong to interface itself for factories/utilities, cannot access instance `this`; default methods belong to instances and provide inheritable behavior using instance state or other abstract methods.

---

### End Notes
Use default methods judiciously: keep them thin, focused, and side-effect minimal. Let static methods serve clear creation and utility roles to enhance cohesion.
