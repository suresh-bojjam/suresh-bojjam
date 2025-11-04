# Java Sealed Classes (Java 17 & 20) – In-Depth Training

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

### 1.1 Motivation
Sealed classes control which other classes or interfaces may extend or implement them. They improve domain modeling, enable exhaustive pattern matching, and allow compiler optimizations by restricting hierarchies. Introduced as preview in Java 15, finalized in Java 17. Java 20 integrates with pattern matching for switch (preview).

### 1.2 Syntax Overview
```java
public sealed interface Shape permits Circle, Rectangle, Square {}
public final class Circle implements Shape { /* ... */ }
public non-sealed class Rectangle implements Shape { /* ... */ }
public final class Square implements Shape { /* ... */ }
```

### 1.3 Permitted Subclasses
Declared using `permits` clause OR discovered implicitly if in same file (for classes without `permits`). All permitted classes must directly extend/implement and compile together.

### 1.4 Modifiers Available
| Modifier | Meaning | Further Extension |
|----------|---------|-------------------|
| `final` | Cannot be subclassed | No |
| `sealed` | Restricted set of subclasses | Only permitted |
| `non-sealed` | Opens hierarchy past this point | Yes (unrestricted) |

### 1.5 ASCII Hierarchy Diagram
```
           +------------------+
           |   sealed Shape   |
           +---------+--------+
             permits /  \
                 /      \
         +------+        +------------------+
         |Circle|        |   Rectangle      | (non-sealed -> further subclasses)
         +------+        +------------------+
                            \
                             +------------------+
                             | FancyRectangle   | (extends Rectangle)
                             +------------------+
         +--------+
         | Square |
         +--------+
```

### 1.6 Interplay with Pattern Matching
Sealed hierarchies allow `switch` over a sealed type to be exhaustive (Java 17 pattern matching for switch preview in 20):
```java
static String describe(Shape s) {
    return switch (s) {
        case Circle c -> "Circle";
        case Square sq -> "Square";
        case Rectangle r -> "Rectangle";
    }; // exhaustive because Shape's permitted classes known
}
```

### 1.7 Records & Sealed Types
Records are often used as leaf subclasses in sealed hierarchies for immutable data carriers.
```java
public sealed interface Command permits CreateUser, DeleteUser {}
public record CreateUser(String name) implements Command {}
public record DeleteUser(String id) implements Command {}
```

### 1.8 Access & Packages
Permitted subclasses must be in same module (or package constraints per access rules). Typically placed in same package for friend-like grouping.

### 1.9 Exercise Set (Basics)
1. Define a sealed interface `Vehicle` with `Car`, `Bike` final, and `Truck` non-sealed.
2. Create exhaustive switch over `Vehicle`.
3. Add a record variant for electric car.
4. Show a method returning description via pattern matching.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Rules for Sealed Hierarchies
- Permitted subclasses must explicitly declare one of: `final`, `sealed`, `non-sealed`.
- If omitted, compilation error.
- Classes must reside in same module (or same package if unnamed module) for reliable enforcement.

### 2.2 Implicit Permits
If all subclasses are nested in same file, `permits` clause may be omitted; compiler infers.

### 2.3 Migration Strategy
Refactor large inheritance chains into sealed cores + non-sealed extension boundaries to reduce unbounded polymorphism.

### 2.4 Performance Considerations
JIT may optimize virtual dispatch when set of subclasses is known and stable, enabling devirtualization and more efficient pattern matching.

### 2.5 Combining with Pattern Matching for switch (Java 20 Preview)
```java
static int area(Shape s) {
    return switch (s) {
        case Circle c -> (int) (Math.PI * c.radius() * c.radius());
        case Square sq -> sq.side() * sq.side();
        case Rectangle r -> r.width() * r.height();
    };
}
```

### 2.6 Nested Sealed Hierarchies
```java
public sealed interface Expr permits Val, Op {
    record Val(int value) implements Expr {}
    sealed interface Op extends Expr permits Add, Mul {}
    record Add(Expr left, Expr right) implements Op {}
    record Mul(Expr left, Expr right) implements Op {}
}
```

### 2.7 Non-Sealed Escape Hatches
Use `non-sealed` to permit uncontrolled extension beyond a specific boundary while preserving sealing above.

### 2.8 Reflection & Sealed Status
`Class::isSealed()` and `getPermittedSubclasses()` (Java 17) expose sealing metadata.
```java
Class<?> clazz = Shape.class;
if(clazz.isSealed()) {
    for(ClassDesc cd : clazz.getPermittedSubclasses()) {
        System.out.println(cd.displayName());
    }
}
```

### 2.9 Security & Domain Modeling
Restrict external code from introducing unauthorized polymorphic types; improves reliability in plugin architectures.

### 2.10 Sealed vs Enum
Sealed gives richer data & methods (each subclass full class/record), while enum restricts to constants with optional constructors. Sealed allows hierarchical variation and additional fields per subclass.

### 2.11 Evolution & Backward Compatibility
Adding new permitted subclass requires updating `permits` clause; existing exhaustive switches may need revisiting (compiler warnings). Provide default branch until all code switched to pattern matching.

### 2.12 Exercise Set (In-Depth)
1. Create nested sealed hierarchy for arithmetic expressions including value, add, multiply, negate.
2. Use reflection to list permitted subclasses.
3. Demonstrate introducing a `non-sealed` intermediate extension point.
4. Implement exhaustive switch returning evaluated result.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Sealed type mandates permitted subclasses be? | Abstract | Listed or inferred | Only final | In any package | B | Must be explicitly listed or implicitly same file. |
| 2 | Permitted subclass must declare? | Any access | No modifier | final/sealed/non-sealed | synchronized | C | One of the three to satisfy sealing rules. |
| 3 | `non-sealed` keyword effect? | Removes class | Makes class abstract | Opens further subclassing | Disallows interfaces | C | Allows unrestricted extension below. |
| 4 | Benefit of sealed with pattern matching? | Slower dispatch | Exhaustiveness checking | More boilerplate | No impact | B | Compiler ensures all cases covered. |
| 5 | Which is NOT a sealing modifier? | sealed | final | static | non-sealed | C | static is unrelated. |
| 6 | Relationship to enums? | Same feature | Superset enabling data-rich variants | Replaced enums | Only for primitives | B | Sealed enables richer variant data. |
| 7 | Reflection method to check sealed? | `isSealed()` | `sealed()` | `hasPermits()` | `isFinal()` | A | isSealed available. |
| 8 | Adding subclass to sealed interface requires? | Restart JVM | Update permits clause | Mark child static | Nothing | B | Permits must list new subclass. |
| 9 | Exhaustive switch fails if? | Default present | All cases handled | New subclass added not handled | Using records | C | New subclass requires case addition. |
| 10 | `non-sealed` applied to? | Methods | Packages | Classes/Interfaces | Fields | C | Keyword for classes/interfaces. |
| 11 | Sealed hierarchy extends across modules if? | Module exports & accessible | Always | Prohibited | Only with reflection | A | Standard access rules apply. |
| 12 | Pattern matching synergy mainly with? | switch statements | for loops | while loops | try blocks | A | switch supports pattern matching. |
| 13 | Which combination is valid? | sealed + synchronized | sealed + permits | final + permits | non-sealed + enum | B | permits pairs with sealed. |
| 14 | Reason to choose `non-sealed`? | Optimize memory | Provide extension point | Remove superclass methods | Enable GC | B | Creates open branch. |
| 15 | Records in sealed hierarchies provide? | Mutable state | Simplified immutable variants | Performance penalty | Unsealed behavior | B | Records concise immutable carriers.

### MCQ Extension Exercise
Add 2 MCQs: one about reflection API, one about nested sealed hierarchies.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Basic Sealed Hierarchy
Implement sealed `Vehicle` with final leaves and one non-sealed branch.

### Exercise 4.2: Pattern Matching Switch
Write exhaustive switch over `Command` sealed hierarchy returning status strings.

### Exercise 4.3: Expression Evaluator
Build sealed expression algebra; evaluate using switch.

### Exercise 4.4: Reflection Listing
Print permitted subclasses of a sealed interface.

### Exercise 4.5: Introducing Non-Sealed Extension
Convert one final subclass to non-sealed and add new subclass; adapt switch.

### Exercise 4.6: Record Variants
Add record variants to existing sealed type (e.g., shapes with data).

### Exercise 4.7: Migration from Open Hierarchy
Refactor legacy base class into sealed with limited permitted subclasses.

### Exercise 4.8: Security Modeling
Model authorization result with sealed outcomes: `Allowed`, `Denied(reason)`, `RequiresMFA`.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Missing Modifier in Permitted Subclass
```java
public sealed interface S permits A {}
class A implements S {} // Compile error: A must be final, sealed, or non-sealed
```

### 5.2 Non-Permitted Implementation
```java
public sealed interface S permits A {}
final class A implements S {}
final class B implements S {} // Compile error: B not permitted
```

### 5.3 Permit Clause Mismatch
```java
public sealed class Base permits Child {}
final class Other extends Base {} // Error: Other not in permits
```

### 5.4 Using Sealed Without Listing Subclasses (Separate Files)
Omitting permits when subclasses not in same file leads to compilation error.

### 5.5 Switch Exhaustiveness Failure
Adding new subclass but forgetting to handle in switch -> compile error (pattern matching switch) or runtime logic gap.

### 5.6 Reflection Misuse
Assuming `getPermittedSubclasses()` returns loaded classes (it returns `ClassDesc` not `Class<?>` objects). Need to resolve names if needed.

### 5.7 Exercise: Identify Problems
```java
public sealed interface Result permits Ok, Err {}
final class Ok implements Result {}
class Err implements Result {}
```
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Algebraic Data Modeling
Use sealed interfaces + records to emulate ADTs (sum types) enabling exhaustive reasoning.

### 6.2 Versioning Strategy
Use `non-sealed` for experimental branch while stabilizing other variants.

### 6.3 Pattern Matching Optimization
Leverage sealed + records allowing concise destructuring (future advanced pattern matching).

### 6.4 Serialization Safety
Restrict deserialization targets by validating permitted subclass names.

### 6.5 Guarded Extension
Use `non-sealed` plus plugins loaded reflectively; verify legitimacy before use.

### 6.6 Combined Hierarchies
Nest sealed sub-hierarchy inside parent sealed interface for refined algebraic decomposition.

### 6.7 Testing Exhaustiveness
Add new leaf & ensure switch compile error prompts test coverage update.

### 6.8 Domain Event Modeling
Represent event types via sealed interface; each record leaf contains payload.

### 6.9 Performance Hint
Sealed hierarchies may help JIT inline leaf calls due to known set.

### 6.10 Exercise: Implement authorization sealed hierarchy with evaluation function producing messages.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Difference between sealed and final?  
Sealed restricts known set of subclasses; final forbids any subclassing.
2. Why introduce non-sealed?  
Provides escape hatch for extension where needed while retaining sealing above.
3. How do sealed classes aid pattern matching?  
Compiler knows exhaustive set enabling completeness checking in switch expressions.
4. Contrast sealed classes with enums.  
Enums fixed constants; sealed classes allow rich per-subclass data & methods, multiple levels.
5. Can you mix records with sealed interfaces?  
Yes; records often used as leaves for concise immutable data.
6. How does reflection report permitted subclasses?  
`getPermittedSubclasses()` returns array of `ClassDesc` descriptors.
7. Strategy for evolving sealed hierarchy?  
Add new subclass & update permits; provide fallback logic until all switches updated.
8. Why place subclasses in same package?  
Simplifies access control and encapsulation; consistent module boundaries.
9. Are sealed classes about performance only?  
No—primarily domain modeling & correctness, performance is secondary benefit.
10. When to choose enum instead?  
Finite set of values without need for distinct data structures or methods per variant.

### Exercise: Explain difference between `non-sealed` and removing sealing entirely.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. Vehicle hierarchy sample code.
2. Exhaustive switch sample.
3. Record variant `record ElectricCar(int batteryKWh) implements Vehicle {}`.
4. Description method uses switch expression.

## Solutions: Module 2 Exercises
1. Expression algebra code snippet provided.
2. Reflection listing demonstration.
3. Non-sealed intermediate example.
4. Evaluation switch returning integer result.

## Solutions: Module 3 MCQ Extension
1. Reflection API MCQ answer: use `isSealed()` + list `getPermittedSubclasses()`.
2. Nested hierarchy MCQ answer: sealed inside sealed allows segmented variant groups.

## Solutions: Module 5 Error Identification (5.7)
Issues: `Err` lacks required modifier (must be final/sealed/non-sealed). All permitted must compile together.

## Solutions: Module 6 Authorization Exercise
Implement hierarchy `sealed interface AuthResult permits Allowed, Denied, RequiresMfa` with records or classes.

## Solutions: Module 7 Exercise
`non-sealed` retains sealing at parent but opens child; removing sealing entirely allows unrestricted subclassing from root.

---

### End Notes
Sealed classes refine inheritance for correctness, enable exhaustive pattern matching, and integrate with modern Java language features (records, pattern matching for switch). Use them to model constrained domains clearly.
