# Record Patterns (Java 21) – Comprehensive Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

Record Patterns (JEP 440/441 synergy) extend pattern matching by allowing deconstruction of record instances directly in `instanceof`, `switch`, and enhanced `for` contexts, providing concise and safe access to components while enabling nested and exhaustive analyses with sealed hierarchies.

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
Prior to record patterns, extracting components of a record required verbose accessor calls. Record patterns provide a declarative way to deconstruct records, improving readability and enabling more expressive structural pattern matching across complex object graphs.

### 1.2 What Is a Record Pattern?
Syntax embedding record component list directly in pattern position:
```java
if (obj instanceof Point(int x, int y)) {
    System.out.println(x + "," + y);
}
```

### 1.3 Basic Record Definition
```java
public record Point(int x, int y) {}
```

### 1.4 Instanceof Deconstruction
```java
Object o = new Point(3,4);
if (o instanceof Point(int a, int b)) {
    System.out.println(a + b); // 7
}
```

### 1.5 Switch with Record Patterns
```java
static String render(Object o) {
    return switch (o) {
        case Point(int x, int y) -> "Point(" + x + "," + y + ")";
        default -> "Unknown";
    };
}
```

### 1.6 Nested Patterns
```java
public record Line(Point start, Point end) {}
Line line = new Line(new Point(1,2), new Point(3,4));
if (line instanceof Line(Point(int x1, int y1), Point(int x2, int y2))) {
    int dx = x2 - x1;
}
```

### 1.7 ASCII Diagram – Deconstruction Flow
```
+---------------------+
|   Record Instance   |
+----------+----------+
           | pattern match
           v
    +--------------+
    | Component 1  | -> variable binding
    +--------------+
    | Component 2  | -> variable binding
    +--------------+
```

### 1.8 Exhaustiveness & Sealed Types
Combining sealed hierarchies with record patterns allows compiler to verify coverage of all cases.

### 1.9 Pattern Matching Semantics
- Match succeeds if object is non-null and of record type.
- Deconstructs components, binding them to pattern variables.
- Variables are in scope only within the guarding block/arm.

### 1.10 Exercise Set (Basics)
1. Create record `Rectangle(int w,int h)` and pattern-match to compute area.
2. Use `instanceof` with `Point` pattern to print sum of coordinates.
3. Nested pattern: define `Box(Point p)` and extract `x`.
4. Switch on `Point` producing formatted output.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Pattern Refinement with Guards
```java
if (obj instanceof Point(int x, int y) && x == y) {
    // diagonal point
}
```

### 2.2 Dominance & Unreachable Arms
```java
switch (o) {
    case Point(int x, int y) -> ...
    case Point(int a, int b) -> ... // unreachable duplicate pattern
}
```
Compiler flags second as unreachable.

### 2.3 Nested & Mixed Patterns with Records + Type Patterns
```java
if (shape instanceof Rectangle(int w, int h) && w > h) { /* landscape */ }
```

### 2.4 Record Patterns in Enhanced For (Preview/Evolving Concept)
Potential usage (conceptual):
```java
for (Point(int x, int y) : points) {
    // uses pattern variable binding per element
}
```

### 2.5 Binding vs Reuse of Accessors
Under-the-hood accessor methods supply component values; pattern binds them once—no repeated calls in user code.

### 2.6 Nested Sealed Hierarchy Example
```java
sealed interface Shape permits Point, Rectangle {}
public record Rectangle(int w,int h) implements Shape {}
public record Point(int x,int y) implements Shape {}

static int area(Shape s) {
    return switch (s) {
        case Rectangle(int w,int h) -> w*h;
        case Point(int x,int y) -> 0; // point has no area
    };
}
```

### 2.7 Deconstruction with Additional Expressions
```java
if (obj instanceof Point(int x, int y) && (x + y) % 2 == 0) {
    // even sum point
}
```

### 2.8 Variable Naming & Shadowing
Pattern variables cannot conflict with existing scoped variables in same pattern location.

### 2.9 Performance Considerations
Pattern matching introduces minimal overhead—similar to calling component accessors once. Eliminates repeated manual calls.

### 2.10 Migration Strategy
Refactor verbose access code:
```java
if (p instanceof Point) {
    int x = ((Point)p).x();
    int y = ((Point)p).y();
}
```
To:
```java
if (p instanceof Point(int x, int y)) { /* use x,y */ }
```

### 2.11 Advanced Nested Pattern – Tree Example
```java
sealed interface Node permits Leaf, Branch {}
record Leaf(int value) implements Node {}
record Branch(Node left, Node right) implements Node {}

int sum(Node n) {
    return switch (n) {
        case Leaf(int v) -> v;
        case Branch(Leaf(int lv), Leaf(int rv)) -> lv + rv;
        case Branch(Node l, Node r) -> sum(l) + sum(r); // fallback
    };
}
```

### 2.12 Guarded Switch Arms
```java
return switch (p) {
    case Point(int x, int y) when x == y -> 1; // diagonal
    case Point(int x, int y) -> 0;
    default -> -1;
};
```

### 2.13 Exercise Set (In-Depth)
1. Add guard to pattern recognizing diagonal Points.
2. Implement `area` with sealed shapes.
3. Write tree sum using nested record patterns.
4. Demonstrate unreachable duplicate pattern in switch.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Record patterns primarily enable? | Direct mutation | Structural deconstruction | Serialization | Reflection-only access | B | Provide declarative component access. |
| 2 | `Point(int x,int y)` in instanceof does what? | Creates new Point | Deconstructs if instance is Point | Always false | Casts silently | B | Matches and binds fields. |
| 3 | Nested pattern example? | `Line(Point p)` | `Line(Point(int x,int y), Point(int a,int b))` | `Point()` | `Point{}` | B | Shows nested decomposition. |
| 4 | Unreachable duplicate pattern arm leads to? | Runtime error | Compile warning only | Compile-time error | Silent acceptance | C | Compiler rejects unreachable arm. |
| 5 | Guard placement in switch? | After arrow | Using `when` keyword | Using `if` outside | Not allowed | B | Use `when` for guarded arm. |
| 6 | Scope of bound variables? | Whole method | Pattern arm/block only | Global | Class-wide | B | Limited to pattern scope. |
| 7 | Performance overhead vs manual access? | Higher | Lower due to caching | Same order | Unpredictable | C | Similar to accessor calls. |
| 8 | Combining sealed interfaces with patterns yields? | Less safety | Exhaustiveness checking | Removal of casting | Slower code | B | Compiler ensures coverage. |
| 9 | Pattern fails when object is? | Null or not matching type | Any object | Primitive only | Same type | A | Must be non-null & correct type. |
| 10 | Record patterns reduce? | Boilerplate casts/accessors | Memory usage drastically | JIT compilation | Reflection needs always | A | Eliminate repeated accessor boilerplate. |
| 11 | Guarded pattern condition evaluation timing? | Before deconstruction | After binding variables | At runtime before binding | Never | B | Variables bound then guard evaluated. |
| 12 | Shadowing in pattern variables? | Allowed | Forbidden | Causes runtime exception | Always renamed | B | Compiler forbids conflicts. |
| 13 | Fallback arm in sealed switch needed? | Always | Only if not exhaustive | Never | Only for records | B | Exhaustiveness removes need; else include default. |
| 14 | Pattern matching supports further? | Method overriding | Structural recursion | Dynamic proxies | GC tuning | B | Enables recursion over structure. |
| 15 | Tree sum example shows? | Side effects | Nested pattern with fallback | Primitive boxing | Security issue | B | Patterns nested with a fallback arm.

### MCQ Extension
16. Using `when` guards allows? (A: Additional predicate filtering)  
17. Variable scope limited to? (A: Pattern arm/block)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Basic Deconstruction
```java
Point p = new Point(5,7);
if (p instanceof Point(int x, int y)) {
    System.out.println(x * y);
}
```

### 4.2 Guarded Pattern
```java
if (p instanceof Point(int x, int y) && x == y) {
    System.out.println("Diagonal: " + x);
}
```

### 4.3 Switch Exhaustiveness
```java
sealed interface Shape permits Point, Rectangle {}
record Rectangle(int w,int h) implements Shape {}
record Point(int x,int y) implements Shape {}

String classify(Shape s) {
    return switch (s) {
        case Point(int x,int y) -> "Point";
        case Rectangle(int w,int h) -> "Rectangle";
    };
}
```

### 4.4 Nested Record Pattern
```java
record Line(Point start, Point end) {}
Line line = new Line(new Point(1,2), new Point(3,4));
if (line instanceof Line(Point(int sx,int sy), Point(int ex,int ey))) {
    System.out.println("dx=" + (ex - sx));
}
```

### 4.5 Tree Sum with Patterns
```java
sealed interface Node permits Leaf, Branch {}
record Leaf(int value) implements Node {}
record Branch(Node left, Node right) implements Node {}

int sum(Node n) {
    return switch (n) {
        case Leaf(int v) -> v;
        case Branch(Leaf(int lv), Leaf(int rv)) -> lv + rv;
        case Branch(Node l, Node r) -> sum(l) + sum(r);
    };
}
```

### 4.6 Guarded Switch Arm
```java
int classify(Point p) {
    return switch (p) {
        case Point(int x, int y) when x == y -> 1;
        case Point(int x, int y) -> 0;
    };
}
```

### 4.7 Migration Example
```java
// Before
if (p instanceof Point) {
    int x = ((Point)p).x();
    int y = ((Point)p).y();
}
// After
if (p instanceof Point(int x,int y)) {
    // use x, y
}
```

### 4.8 Exercise Set (Practical)
1. Add guard checking `x + y > 10`.
2. Extend `Shape` with `Circle(int r)` and handle in switch.
3. Add nested pattern for `Line(Point(int x1,int y1), Point(int x2,int y2))` computing length.
4. Tree sum adding multiplication for leaves > 10.
5. Demonstrate unreachable duplicate pattern in switch.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Unreachable Pattern Arm
Duplicate record pattern causes compile-time error.

### 5.2 Non-Exhaustive Switch without Default
If sealed hierarchy extended accidentally or preview mismatch -> compilation error requiring default.

### 5.3 Incorrect Component Types
Pattern component types must match record definition; mismatch -> compile error.

### 5.4 Variable Name Conflict
Using variable names already in scope invalidates pattern.

### 5.5 Guard Expression Null Access
Guard referencing null variable outside of bound scope leads to NPE at runtime.

### 5.6 Exercise Set (Errors)
Identify: duplicate pattern arm, non-exhaustive, component type mismatch.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Structural Recursion with Patterns
Use nested patterns in switch for succinct recursive algorithms.

### 6.2 Guard Placement for Optimization
Place cheap guard conditions early to short-circuit evaluation.

### 6.3 Domain Modeling
Combine records + sealed for algebraic data types enabling exhaustive & concise processing.

### 6.4 Fallback Arms
Provide fallback arm catching complex nested patterns, after simpler direct cases.

### 6.5 Pattern Ordering Strategy
Put more specific patterns before generic ones to avoid dominance issues.

### 6.6 Simplifying Mappers
Pattern matching transforms data objects to DTOs with minimal boilerplate.

### 6.7 Debugging Patterns
Use pattern variable logging to trace matching flow.

### 6.8 Performance Micro-Optimizations
Avoid heavy computations in guards; compute once and reuse variable.

### 6.9 Refactoring Legacy Code
Replace cascaded `if (instanceof)` and casts with unified switch + record patterns.

### 6.10 Exercise Set (Techniques)
Refactor legacy visitor pattern into switch using record patterns showing reduced boilerplate.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. Explain how record patterns integrate with sealed types.
2. Difference between type patterns and record patterns.
3. How guards enhance pattern semantics.
4. What is dominance in pattern matching?
5. Why do record patterns aid readability?
6. Discuss performance implications.
7. Migration challenges from older codebases.
8. Provide example of nested pattern in algorithm.
9. How exhaustive switches reduce bugs.
10. Potential pitfalls when adding new variants.

### Exercise
Design a small expression evaluator using record patterns and sealed hierarchy.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
Area from Rectangle pattern; sum of coordinates; nested Box deconstruction; switch rendering.

## Solutions: Module 2 Exercises
Diagonal guard usage; area sealed shapes implementation; tree sum recursion; unreachable duplicate demonstration.

## Solutions: Module 3 MCQ Extension
16: Guards add extra predicate filtering; 17: Scope limited to arm/block.

## Solutions: Module 4 Practical
Guard `x + y > 10`; extend with Circle variant; line length using distance formula; modify tree sum for special leaf multiplication; unreachable duplicate pattern snippet.

## Solutions: Module 5 Errors
Duplicate pattern leads compile error; not exhaustive requires default; mismatch types corrected.

## Solutions: Module 6 Techniques
Visitor refactor using switch-case patterns reduces boilerplate; ordering prevents dominance conflicts.

## Solutions: Module 7 Exercise
Expression evaluator: sealed hierarchy (Const, Add, Mul); switch with record patterns computing result recursively.

---

### End Notes
Record patterns are a powerful addition to Java's pattern matching arsenal, enabling clearer, safer, and more concise structural decomposition. Combined with sealed hierarchies and guards they facilitate expressive, maintainable domain logic.
