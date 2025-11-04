# Java Records (Java 17) – In-Depth Training

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

### 1.1 Purpose
Records provide a concise syntax for declaring shallowly immutable data-carrier classes. They automatically supply canonical constructor, accessors, `equals`, `hashCode`, and `toString` based on their components.

### 1.2 Syntax Overview
```java
public record Point(int x, int y) {}
```
Equivalent boilerplate in a traditional class would involve fields, constructor, getters, equals, hashCode, toString.

### 1.3 Shallow Immutability
Record components are final; object references inside can still be mutable. Ensure deep immutability manually.

### 1.4 Generated Members
- Accessors: component name methods (`x()`, `y()`)
- Canonical constructor: parameters match component list
- `equals`/`hashCode`: structural comparison
- `toString`: `Point[x=..., y=...]`

### 1.5 ASCII Structure Diagram
```
+-----------------------------------------+
| record Point(int x, int y)              |
|-----------------------------------------|
| Components: x:int, y:int                |
| Auto: constructor(x,y)                  |
| Auto: accessors x(), y()                |
| Auto: equals(), hashCode(), toString()  |
+-----------------------------------------+
```

### 1.6 Compact Constructors
```java
public record Range(int start, int end) {
    public Range { // compact canonical ctor
        if (start > end) throw new IllegalArgumentException("start>end");
    }
}
```

### 1.7 Explicit Canonical Constructor
```java
public record User(String name, int age) {
    public User(String name, int age) { // explicit canonical
        this.name = name.trim();
        this.age = age < 0 ? 0 : age;
    }
}
```
(Note: Must assign all components directly; cannot call another constructor first.)

### 1.8 Static & Instance Members
Records can declare static fields/methods and instance methods. Instance fields beyond components are discouraged (only implicitly final component fields allowed; additional instance fields are effectively prohibited).

### 1.9 Sealed Records
Records can implement sealed interfaces or be permitted subclasses.
```java
public sealed interface Shape permits Circle {}
public record Circle(double r) implements Shape {}
```

### 1.10 Exercise Set (Basics)
1. Create `record Person(String name, int age)`.
2. Add validation using compact constructor.
3. Override `toString` adding masking for age.
4. Demonstrate accessor usage.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Serialization Considerations
Records preserve component order and names; `readResolve`/`writeReplace` can still customize. Avoid relying solely on `toString` for persistence.

### 2.2 Pattern Matching Synergy
Records integrate with pattern matching for `instanceof` / switch (preview features): deconstruction pattern `Point(int x, int y)`.
```java
static int manhattan(Point p) {
    return switch (p) { case Point(int x, int y) -> Math.abs(x) + Math.abs(y); };
}
```

### 2.3 Nesting & Composition
Use records as components of larger aggregates; caution with deep nesting for readability.

### 2.4 Validation Strategies
Compact constructor vs factory method vs static validator. Keep invariants enforced at creation for consistency.

### 2.5 Implementing Interfaces
Records implement interfaces naturally:
```java
public interface Identified { String id(); }
public record User(String id, String email) implements Identified {}
```

### 2.6 Equality Semantics vs Traditional Classes
Records define structural equality across components; classes may adapt custom logic.

### 2.7 Performance Considerations
Records reduce boilerplate; JIT may optimize due to straightforward layout. Still objects (not value types yet—Project Valhalla future). Avoid excessive creation in hot loops without need.

### 2.8 Encapsulation Limits
Components are exposed via accessors; cannot hide internal representation like private fields. Use records only when data transparency acceptable.

### 2.9 Mutability Workarounds
To "modify" a record, create a new instance.
```java
Point p2 = new Point(p.x()+1, p.y());
```

### 2.10 Local Records
Inside methods for narrow-scope modeling.
```java
void process(List<String> lines) {
    record LineInfo(int index, String content) {}
    List<LineInfo> infos = IntStream.range(0, lines.size())
        .mapToObj(i -> new LineInfo(i, lines.get(i))).toList();
}
```

### 2.11 Reflection Support
`Class.isRecord()` and `getRecordComponents()` provide metadata.
```java
RecordComponent[] comps = Point.class.getRecordComponents();
for (RecordComponent rc : comps) System.out.println(rc.getName()+" : "+rc.getType());
```

### 2.12 Generic Records
```java
public record Pair<A,B>(A first, B second) {}
```

### 2.13 Migration Strategy
Replace verbose POJOs where identity is defined by all fields and immutability desired. Avoid if behavior-rich or invariants complex.

### 2.14 Exercise Set (In-Depth)
1. Implement generic `record Result<T>(T value, boolean success)`.
2. Use pattern matching to extract components.
3. Create local record to summarize list statistics.
4. Reflect over a record printing component names.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Records are? | Mutable data carriers | Shallowly immutable | Value types | Enums | B | Components final; shallow immutability. |
| 2 | Auto-generated method NOT included? | equals | hashCode | clone | toString | C | clone not auto-generated. |
| 3 | Constructor style enforcing invariants? | Compact canonical | Synthetic | Secondary | Reflection | A | Compact constructor uses component list implicitly. |
| 4 | Accessor naming convention? | getX() | x() | fetchX() | fieldX() | B | Accessor named after component. |
| 5 | Record components must be? | private | final implicitly | transient | volatile | B | They are final and implicitly public accessor. |
| 6 | Pattern matching deconstruction form? | `Point.point(x,y)` | `Point(int x, int y)` | `new Point(x,y)` | `Point[x,y]` | B | Deconstruction pattern syntax. |
| 7 | Records can extend? | Other classes | Object only | Interfaces only | Enums | B | Implicitly extend Object, cannot extend other classes. |
| 8 | Adding extra instance field allowed? | Yes freely | Only static | Not (only components) | Only private | C | Only components define state. |
| 9 | `isRecord()` returns? | true for any class | true for record types | false always | true for enums | B | Identifies record types. |
| 10 | Which modifies components? | Reassign inside method | Use setter | Create new instance | Reflection only | C | Must construct new instance. |
| 11 | Best POJO replacement scenario? | Complex inheritance | Data carrier with all-field identity | Frequent field mutation | Hidden representation needed | B | Records suit transparent data carriers. |
| 12 | Generic record example? | `record Box<T>(T v)` | `class Box<T>(T v)` | `enum Box<T>` | `record<T> Box(T v)` | A | Generic syntax as shown. |
| 13 | Overriding accessor allowed? | Yes (with same signature) | No | Only with annotation | Only if static | A | Allowed but should preserve contract. |
| 14 | Deep immutability guaranteed? | Yes always | No (components may be mutable objects) | Only with primitives | Only with final classes | B | Shallow only. |
| 15 | Reflection component retrieval method? | `getComponents()` | `getRecordComponents()` | `components()` | `getFields()` | B | API provided.

### MCQ Extension Exercise
Add 2 MCQs: one about local records, one about sealed interfaces with records.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Validation with Compact Constructor
Create `Range` record ensuring start <= end.

### Exercise 4.2: Pattern Matching Switch
Use switch expression with record deconstruction.

### Exercise 4.3: Generic Pair
Implement `Pair<A,B>` and use in grouping operation.

### Exercise 4.4: Local Record Stats
Compute min, max, average using local record.

### Exercise 4.5: Immutable Update
Demonstrate updating a point's x coordinate by creating new instance.

### Exercise 4.6: Reflection Printer
Print names and types of components for a record.

### Exercise 4.7: Record in Sealed Hierarchy
Integrate record in sealed `Event` type.

### Exercise 4.8: Masked toString Override
Override `toString` to hide sensitive component.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Incorrect Field Assignment in Canonical Constructor
Trying to use `this.x = ...` after calling another constructor; must assign directly within canonical.

### 5.2 Missing Component Assignment
Failing to assign all components explicitly when writing explicit canonical constructor.

### 5.3 Adding Mutable Field
Attempting to add non-static field besides components results in compilation error.

### 5.4 Serialization Assumptions
Assuming `toString` stable for parsing causes fragile code.

### 5.5 Pattern Matching Incomplete (Preview)
Using record pattern without enabling preview features (in older JDK) leads to compilation error.

### 5.6 Exercise: Identify Errors
```java
public record Bad(int a, int b) {
    public Bad(int a) { this(a, 0); }
    public Bad(int a, int b) { a = a; b = b; }
}
```

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 DTO Replacement
Use records for REST responses and value objects.

### 6.2 Defensive Copying
For mutable components (e.g., arrays) perform copy in constructor.
```java
public record Data(byte[] payload) {
    public Data { payload = payload.clone(); }
}
```

### 6.3 Builder Alternative
Use static factory for optional normalization.
```java
public record User(String name, int age) {
    public static User of(String name, int age) { return new User(name.trim(), Math.max(age,0)); }
}
```

### 6.4 Deconstruction Utilities
Pattern matching reduces verbosity in destructuring.

### 6.5 Functional Composition
Records pair naturally with functional pipelines for grouping and transformation results.

### 6.6 Caching & Memoization Keys
Structural `equals` & `hashCode` ideal for map keys.

### 6.7 Domain Modeling with Sealed + Records
Use sealed interface for variants, record leaves for data semantics.

### 6.8 Local Records for Intermediate States
Clarify ephemeral transformations instead of ad-hoc maps.

### 6.9 Overriding Accessor for Derived Value
Compute trimmed string or normalized form.

### 6.10 Exercise: Implement record with defensive copies and custom accessor normalization.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Why records introduced?  
Reduce boilerplate for transparent data carriers with semantic clarity.
2. Differences vs Lombok @Data?  
Native language feature, enforced immutability, no annotation processing.
3. How to enforce invariant?  
Compact canonical constructor or static factory.
4. Are records value types?  
No; still reference types until Valhalla introduces value objects.
5. When not to use records?  
Need encapsulation, complex invariants, or evolving stateful behavior.
6. How do records help pattern matching?  
Support deconstruction patterns enabling more expressive switch/instanceof.
7. Can records be generic?  
Yes standard generic syntax.
8. Deep immutability concerns?  
Mutable components require defensive copy.
9. Interaction with sealed types?  
Records commonly used as leaves, enabling exhaustive reasoning.
10. Reflection capabilities?  
`isRecord()`, `getRecordComponents()` return metadata for introspection.

### Exercise: Explain difference between compact constructor and explicit canonical constructor.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. `record Person(String name,int age){}`
2. Compact constructor validation example.
3. Override toString adding masking.
4. Accessors: `person.name()` etc.

## Solutions: Module 2 Exercises
1. `record Result<T>(T value, boolean success){}`.
2. Pattern matching extraction sample.
3. Local record usage shown.
4. Reflection printing code sample.

## Solutions: Module 3 MCQ Extension
1. Local records limited scope advantage.
2. Sealed interface synergy answer.

## Solutions: Module 5 Error Identification (5.6)
Issues: Missing full component assignment in explicit canonical, improper delegation; canonical must assign `this.a = a; this.b = b;` after normalization.

## Solutions: Module 6 Defensive Copy Exercise
Sample Data record shown earlier.

## Solutions: Module 7 Exercise
Compact constructor implicitly takes component params; explicit canonical declares parameter list manually; both must assign components directly.

---

### End Notes
Use records where transparency, immutability, and structural equality align with domain intent. Pair with sealed interfaces and pattern matching for expressive, safe modeling.
