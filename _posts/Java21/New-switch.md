# Java 21 Pattern Matching for switch & Switch Expressions – In-Depth Training

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

### 1.1 Evolution of switch
- Pre Java 7: `switch` over primitives, `char`, `String` added in Java 7.
- Java 12+: switch expressions (preview) -> Java 14 standard: yield value.
- Java 21: Pattern Matching for switch (JEP 441) extends type test patterns and guarded patterns for safer exhaustive type analysis.

### 1.2 Switch Expression Recap
```java
int result = switch (day) {
    case MONDAY, FRIDAY -> 6;
    case TUESDAY -> 7;
    default -> {
        int calc = day.ordinal();
        yield calc;
    }
};
```

### 1.3 Pattern Matching for switch
Allows patterns (type + variable binding) directly in case labels with optional guards:
```java
String format(Object o) {
    return switch (o) {
        case null -> "null"; // null selector case
        case String s -> "String:" + s;
        case Integer i && i > 0 -> "Positive int:" + i;
        case Integer i -> "Int:" + i;
        case Long l -> "Long:" + l;
        default -> "Other:" + o.getClass().getSimpleName();
    };
}
```

### 1.4 Flow Analysis & Exhaustiveness
Compiler ensures that for sealed hierarchies all possible subtypes are covered or a default case supplied—improves safety similar to enum exhaustive switches.

### 1.5 ASCII Concept Diagram
```
          +------------------------------+
          |        switch (value)       |
          +---------------+--------------+
                          |
                  Type + Guard Patterns
                          |
     +-----------+   +--------------+   +--------------+
     | case T t  |   | case U u && g|   | case null    |
     +-----------+   +--------------+   +--------------+
                          |
                     Exhaustive Analysis
```

### 1.6 Pattern Categories
| Pattern | Example | Purpose |
|---------|---------|---------|
| Type Pattern | `case String s` | Test & bind if instance of type |
| Guarded Pattern | `case Integer i && i > 0` | Add boolean condition after type match |
| Null Case | `case null` | Explicit null handling |
| Default | `default` | Catch-all when non-exhaustive |

### 1.7 Dominance & Unreachable Cases
A more general pattern preceding a specific one can make the latter unreachable (compile-time error). Order matters for overlapping patterns.

### 1.8 Exercise Set (Basics)
1. Convert traditional `instanceof` + cast chain to pattern matching switch.
2. Add a null case to a switch expression.
3. Use a guarded pattern for positive integers.
4. Demonstrate an unreachable pattern error by ordering wrongly (then fix).

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Exhaustiveness with Sealed Hierarchies
```java
sealed interface Shape permits Circle, Square {}
record Circle(double r) implements Shape {}
record Square(double s) implements Shape {}

String describe(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle r=" + c.r();
        case Square s -> "Square s=" + s.s();
    }; // exhaustive - no default needed
}
```
If a new permitted subtype is added, compiler flags non-exhaustive switch.

### 2.2 Guarded Pattern Semantics
Guard evaluated only after successful type match; variable in guard is already typed.
```java
switch (obj) {
    case String s && s.length() > 5 -> ...
    case String s -> ...
}
```

### 2.3 Handling null Safely
Explicit `case null` avoids `NullPointerException` from dereferencing. Null is distinct selector and must precede other patterns or appear among them—no automatic inclusion.

### 2.4 Dominance Rules Detailed
A pattern P dominates Q if every selector matched by Q also matched by P earlier with no guard difference. Compiler rejects unreachable Q.

### 2.5 Fall-Through Elimination
Switch expressions disallow traditional fall-through unless using multiple labels delimited by commas; each arrow arm isolated.

### 2.6 `yield` vs Arrow
Arrow form implicitly yields; block form uses `yield` for value.
```java
int size = switch (o) {
    case String s -> s.length();
    case int[] arr -> { yield arr.length; }
    default -> 0;
};
```

### 2.7 Pattern Matching + Enum Cases Mixed
You can combine constant labels and patterns:
```java
enum Kind { TEXT, NUMBER }
Object val = ...;
String out = switch (val) {
    case Kind.TEXT -> "Enum TEXT";
    case String s -> s;
    default -> "Other";
};
```

### 2.8 Flow Scoping
Variable bound in pattern available only in its arm scope. Cannot leak outside.

### 2.9 Performance Considerations
Pattern matching adds minimal overhead; compiler generates type checks similar to `instanceof`. Exhaustive sealed switches may allow optimized dispatch.

### 2.10 Migration Strategy
Refactor chains:
```java
if (o instanceof String s) { ... }
else if (o instanceof Integer i) { ... }
```
Into single switch for clarity & exhaustiveness.

### 2.11 Example: Nested Patterns
Use pattern switch inside another pattern-based computation for structured data processing.

### 2.12 Exercise Set (In-Depth)
1. Exhaustive switch over a sealed hierarchy with a guard.
2. Show dominance compile error and correct ordering.
3. Combine enum labels and type patterns.
4. Use pattern switch to compute value from heterogeneous list.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Pattern matching for switch introduced standard in? | Java 17 | Java 21 | Java 11 | Java 8 | B | Finalized in Java 21 (JEP 441). |
| 2 | Guarded pattern syntax? | `case T && expr var` | `case T var && expr` | `case var T && expr` | `case T(expr)` | B | Type pattern then var then guard. |
| 3 | Null handling pattern? | `case NULL` | `case null` | `default` | `case (null)` | B | Literal null case. |
| 4 | Unreachable pattern cause? | Missing break | Dominated by previous broader pattern | Too few cases | Using yield | B | Dominance prevents reachability. |
| 5 | Exhaustive switch on sealed type needs default when? | Always | Never | When not all permitted subclasses handled | Only with guards | C | Must cover all subclasses. |
| 6 | Switch expression value return with arrow? | `yield` required | Implicit | Must assign separately | Only for enums | B | Arrow form returns expression. |
| 7 | Pattern variable scope? | Whole method | Entire switch | Only its case arm | Global | C | Scoped to case arm. |
| 8 | Multiple labels combine how? | Chained arrows | Comma-separated | Semicolon-separated | Requires default | B | Use commas. |
| 9 | Guard evaluation timing? | Before type test | After type test success | Random | At compile time | B | Guard after pattern match. |
| 10 | Mixed enum + type patterns allowed? | Yes | No | Only with default | Only with sealed types | A | Can mix constant & type patterns. |
| 11 | `yield` needed when? | Arrow form | Block form returning a value | Single label | Always | B | Block needs explicit yield. |
| 12 | Performance effect vs instanceof chain? | Slower | Similar | Always faster | Allocates extra objects | B | Comparable checks. |
| 13 | Which invalid pattern? | `case String s` | `case Integer i && i>0` | `case null` | `case int i` (primitive pattern in switch object context) | D | Primitive patterns apply with switch over primitive; object switch expects boxed or reference types. |
| 14 | Sealed type new subclass effect? | Nothing | Switch stays exhaustive | Compile error for non-exhaustive switch | Runtime exception | C | Forces update. |
| 15 | Default needed when? | Non-exhaustive pattern coverage | Always | Never | Only with null | A | Provided when coverage incomplete.

### MCQ Extension Exercise
Add 2 MCQs: one about dominance, one about guarded pattern runtime behavior.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Format Dispatcher
Implement format function using pattern switch for types: String (len), Integer (hex), Double (scientific), null.

### Exercise 4.2: Positive Integer Guard
Return different string for positive vs non-positive integers using guarded pattern.

### Exercise 4.3: Sealed Shape Area
Compute area with exhaustive switch; add guard for negative radius (return 0).

### Exercise 4.4: Heterogeneous List Processing
Map list of Objects to strings using single pattern switch expression.

### Exercise 4.5: Dominance Demonstration
Create unreachable case then fix ordering.

### Exercise 4.6: Enum + Type Mix
Switch distinguishing Kind enum vs other objects.

### Exercise 4.7: Nested Pattern Switch
Inside outer switch case apply another pattern switch.

### Exercise 4.8: Refactor Legacy Chain
Refactor if-else instanceof chain into pattern switch and compare readability.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Dominated Pattern
```java
switch (o) {
    case Object obj -> "generic";
    case String s -> s; // Compile error: unreachable (dominated)
}
```

### 5.2 Missing Exhaustiveness
Sealed hierarchy switch missing subtype -> compile error requiring default or handling.

### 5.3 Misuse of Guard Syntax
```java
case Integer && i > 0 -> ... // INVALID
```

### 5.4 Null Not Handled
Switch on object with potential null without case null leads to `NullPointerException` when evaluating selector.

### 5.5 Incorrect yield usage
Using `yield` in arrow case or forgetting yield in block case results in compile errors.

### 5.6 Exercise: Identify Errors
```java
sealed interface Animal permits Cat, Dog {}
record Cat(String name) implements Animal {}
record Dog(String name) implements Animal {}
String speak(Animal a) {
    return switch (a) {
        case Animal an -> an.toString();
        case Cat c -> "Meow"; // unreachable
    };
}
```

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Domain Algebra Modeling
Combine sealed interfaces + pattern switch for exhaustive business logic.

### 6.2 Guard Ordering Optimization
Place specific guarded patterns before general ones to avoid dominance issues.

### 6.3 Null Normalization Pattern
Use `case null -> defaultValue` early to centralize null handling.

### 6.4 Logging & Metrics Simplification
Single switch expression aggregates heterogeneous event types.

### 6.5 Declarative Data Transformation
Use pattern switch within stream mapping for clean transformations.

### 6.6 Fallback Pattern Strategy
End with `default -> throw new IllegalStateException(...)` for defensive programming.

### 6.7 Combining Record Deconstruction (Future Enhancements)
Record patterns (Java 21 also includes deconstruction records) integrate with switch for structural matching.
```java
record Point(int x, int y) {}
String pos(Object o) {
    return switch (o) {
        case Point(int x, int y) && x==y -> "Diagonal";
        case Point(int x, int y) -> x + "," + y;
        default -> "Other";
    };
}
```

### 6.8 Error Classification with Patterns
Switch on exception object: specific types/guards (e.g. transient network errors).

### 6.9 Performance Tuning
Prefer single switch over multiple instanceof checks for readability; micro performance similar.

### 6.10 Exercise: Implement exception classifier using pattern switch.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Why pattern matching added to switch?  
Improve expressiveness, reduce boilerplate, enable exhaustive, type-safe branching.
2. How does guarded pattern differ from separate if?  
Single cohesive branch keeps pattern/type binding and condition localized.
3. When to use default vs exhaustive coverage?  
Default used when open hierarchy or unsure of future subtypes; omit when sealed & fully covered.
4. How null handled differently now?  
Explicit `case null` avoids hidden NPE and documents intent.
5. Impact on readability?  
Condenses multi-branch instanceof chains into declarative arms.
6. Performance overhead?  
Comparable to manual instanceof; improved maintainability.
7. Dominance rule purpose?  
Prevents unreachable arms; ensures clarity and correctness.
8. How record patterns complement?  
Allow direct destructuring inside switch for structural logic.
9. Migration risk?  
Reordering arms can introduce dominance errors; tests should catch logic changes.
10. Guard pitfalls?  
Overusing complex guards reduces readability; prefer small simple conditions.

### Exercise: Explain compile-time dominance detection benefits.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
Converted chain, added null case, positive guard, dominance ordering fixed.

## Solutions: Module 2 Exercises
Exhaustive sealed switch with guard, dominance example re-ordered, enum + type mixed, list mapping pattern switch.

## Solutions: Module 3 MCQ Extension
Dominance MCQ answer: earlier broader pattern makes later narrower unreachable. Guard runtime: guard evaluated post type test; if false proceeds to next case.

## Solutions: Module 5 Error Identification (5.6)
`case Animal an` dominates `case Cat c`; remove generic or place Cat first.

## Solutions: Module 6 Exception Classifier Exercise
Pattern switch mapping error types to categories.

## Solutions: Module 7 Exercise
Compile-time dominance detection prevents dead code & future mismatch when adding new patterns.

---

### End Notes
Pattern matching for switch in Java 21 elevates type-driven branching to a first-class feature enabling concise, exhaustive, and safer logic. Combine with records and sealed interfaces for powerful algebraic modeling capabilities.
