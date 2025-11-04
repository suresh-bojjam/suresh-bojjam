# Unnamed Patterns and Variables (Java 21) – Comprehensive Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

Unnamed patterns and variables (often represented by `_`) streamline pattern matching and local variable declarations when the binding name would be unused. This reduces boilerplate and highlights intent: ignoring components, placeholders for future use, and clarifying control flow.

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
In pattern matching (switch/type/record patterns) sometimes certain components are unnecessary. Prior to unnamed patterns developers used throwaway variable names (e.g., `ignored`, `_1`). Unnamed patterns & variables formalize this ignoring concept, improving readability and reducing noise.

### 1.2 Unnamed Variable Declaration
```java
int _ = compute(); // value computed then ignored
```
Used where side-effects matter but name not used.

### 1.3 Unnamed Pattern Usage
Within record patterns:
```java
record Pair(int left, int right) {}
Pair p = new Pair(10, 20);
if (p instanceof Pair(int left, _)) { // ignore right component
    System.out.println(left);
}
```

### 1.4 ASCII Diagram – Ignoring Components
```
+-------------------------------+
| Record: Pair(left=10,right=20)|
+---------+---------------------+
  left -> bind variable 'left'
  right -> ignored by '_'
```

### 1.5 Semantics
- `_` signals compiler: do not introduce a binding accessible later.
- Enhances clarity in nested patterns with many components.
- Multiple `_` allowed to ignore multiple components.

### 1.6 Limitations
- Cannot reuse `_` where a distinct binding is needed for guards referencing that component.
- `_` does not circumvent exhaustiveness checks.

### 1.7 Exercise Set (Basics)
1. Create a record `Triple(int a,int b,int c)` and match ignoring `b` & `c`.
2. Use unnamed variable to capture (ignore) return of logging function.
3. Match a pair and ignore both components.
4. Use `_` in a switch pattern arm.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Nested Ignoring
```java
record Node(int value, Node next) {}
if (node instanceof Node(int v, Node(_, _))) { /* nested ignore */ }
```

### 2.2 Combining Named and Unnamed
```java
if (p instanceof Pair(_ , int right)) {
    System.out.println(right);
}
```

### 2.3 Multiple Unnamed Fields
```java
if (t instanceof Triple(int a, _, _)) { use(a); }
```

### 2.4 Guards with Unnamed Patterns
Cannot reference `_` in guard; must bind if needed.
```java
if (p instanceof Pair(int left, _ ) && left > 0) { /* ok */ }
```

### 2.5 Scope & Shadowing
`_` introduces no variable binding; avoids name collision risks.

### 2.6 Switch Exhaustiveness & `_`
Ignoring components does not alter coverage; pattern matching still requires covering all record variations (if sealed hierarchy context).

### 2.7 Migration Strategy
Replace throwaway identifiers:
```java
if (p instanceof Pair(int x, int ignored)) { use(x); }
```
becomes
```java
if (p instanceof Pair(int x, _)) { use(x); }
```

### 2.8 Large Pattern Readability
In wide records, ignoring rarely used components using `_` improves scanning speed.

### 2.9 Interop with Deconstruction Patterns
Works seamlessly alongside nested record deconstruction.

### 2.10 Performance Considerations
No measurable runtime cost difference; purely syntactic improvement.

### 2.11 Exercise Set (In-Depth)
1. Implement nested ignore on a linked list two nodes deep.
2. Migrate code using `ignored` variables to `_`.
3. Use switch with a pattern ignoring second component.
4. Demonstrate readability improvement in 6-component record.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | `_` in pattern indicates? | Binding variable | Ignoring component | Runtime deletion | Reflection handle | B | `_` signifies ignore. |
| 2 | Using `_` affects exhaustiveness? | Yes | No | Sometimes | Only with guards | B | Coverage unchanged. |
| 3 | Multiple `_` allowed? | No | Yes | Only two | Only in sealed types | B | Can ignore many components. |
| 4 | Guard referencing ignored value? | Allowed | Must bind to use | Compiles silently | Replaced by 0 | B | Need named binding. |
| 5 | Runtime overhead difference? | Higher | Lower | Neutral | Undefined | C | Pure syntax convenience. |
| 6 | Migration benefit? | More boilerplate | Clearer intent | Slower matching | Breaks code | B | Clarifies unused parts. |
| 7 | Shadowing risk with `_`? | High | None | Only in nested | At runtime only | B | No variable created. |
| 8 | Record pattern `Pair(_,_)` does what? | Fails match | Matches any Pair ignoring values | Throws exception | Partial bind | B | Accepts any Pair. |
| 9 | Using `_` in variable decl `int _ = foo();` means? | Bind usable var | Value ignored intentionally | Compile error | Private var | B | Name is unused placeholder. |
| 10 | `_` can appear in? | Pattern positions & local variable declarations | Class names | Method names | Package names | A | Restricted to those contexts. |
| 11 | Need component for guard? | Use `_` | Bind named variable | Compiler infers | Impossible | B | Must name variable. |
| 12 | Large record readability improved by? | Using `_` for each needed field | Ignoring rarely used fields with `_` | Duplicating names | Extra comments only | B | Reduces noise. |
| 13 | `_` changes matching logic? | Yes drastically | Slightly | Not at all | Causes fallback | C | Only binding removal. |
| 14 | Replace throwaway variable `ignored` with? | `_` | `$` | `__` | Keep same | A | Use `_` idiom. |
| 15 | Pattern `Triple(_,_,_)` matches? | None | Any Triple | Only all zeros | Throws | B | Unconditional Triple match.

### MCQ Extension
16. Ignoring nested node child uses? (A: `_` in nested pattern)  
17. To inspect ignored value later you must? (A: bind it with a name)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Ignoring One Component
```java
record Pair(int left, int right) {}
Pair p = new Pair(5, 10);
if (p instanceof Pair(int left, _)) {
    System.out.println(left); // 5
}
```

### 4.2 Ignoring All Components
```java
if (p instanceof Pair(_, _)) {
    System.out.println("Matched pair; values ignored");
}
```

### 4.3 Nested Ignore in Linked List
```java
record Node(int v, Node next) {}
Node list = new Node(1, new Node(2, new Node(3, null)));
if (list instanceof Node(int head, Node(_, _))) {
    System.out.println(head); // 1
}
```

### 4.4 Switch Ignoring Component
```java
static String render(Pair p) {
    return switch (p) {
        case Pair(int left, _) -> "L=" + left;
    };
}
```

### 4.5 Wide Record Example
```java
record Metrics(int cpu, int mem, int disk, int net, int errors, int warnings) {}
Metrics m = new Metrics(70, 4096, 500, 200, 2, 5);
if (m instanceof Metrics(int cpu, _, _, _, int errors, _)) {
    System.out.println("cpu=" + cpu + " errors=" + errors);
}
```

### 4.6 Ignoring Return Value with Unnamed Variable
```java
int _ = logAndReturn("processed"); // ensure side-effect only
```

### 4.7 Exercise Set (Practical)
1. Ignore second component of Pair printing first.
2. Match Triple ignoring last two values verifying first > 0.
3. Wide record focusing on two metrics only.
4. Nested Node ignoring deeper chain.
5. Use `_` in local variable to ignore result.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Incorrect Usage in Class Name
Attempting `class _ {}` -> compile error (identifier rules).

### 5.2 Referencing Ignored Component
Referencing `_` variable -> compile error: not defined.

### 5.3 Guard Needs Named Variable
```java
if (p instanceof Pair(_, _) && _ > 0) { } // invalid
```

### 5.4 Duplicate Named Binding vs `_`
No compile error with multiple `_`; but wrong expectation referencing them later fails.

### 5.5 Exercise Set (Errors)
Identify: using `_` in guard incorrectly; trying to use `_` as method name; referencing ignored component.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Noise Reduction
Apply `_` aggressively to declutter complex nested patterns.

### 6.2 Intent Signaling
Use `_` to signal review: component intentionally ignored (not oversight).

### 6.3 Progressive Refinement
Begin with `_` placeholders; later replace with names as logic evolves.

### 6.4 Domain-Specific Pattern DSL
Combine with record patterns to emulate ML/functional style decomposition.

### 6.5 Guard Strategy
Only bind what you guard on; ignore the rest.

### 6.6 Migration Script Idea
Automated replacement of variables named `ignored|unused|__` with `_`.

### 6.7 Logging Simplification
Ignore values not needed for logs while binding essential ones only.

### 6.8 Code Review Checklist
Ensure `_` usage intentional; not hiding required validation.

### 6.9 Educational Refactoring
Show students difference between manual casts vs modern pattern ignoring.

### 6.10 Exercise Set (Techniques)
Refactor large visitor pattern converting extraneous binds to `_` for clarity.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. Explain unnamed patterns vs named bindings.
2. How does `_` improve readability?
3. Why doesn't `_` affect exhaustiveness?
4. Provide a migration example.
5. Can `_` be used in guards? Explain.
6. Scope implications of `_`.
7. Interaction with nested patterns.
8. Potential misuse pitfalls.
9. Tooling opportunities (lint rules).
10. How to evolve placeholders into full logic.

### Exercise
Design pattern matching for a deep record where only outer two components matter—use `_` for others.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
Triple match ignoring b,c; unnamed var discarding return; ignoring both Pair components; switch ignoring one component.

## Solutions: Module 2 Exercises
Nested ignore sample list two deep; migration from `ignored` var; switch ignoring second; readability improvement demonstration.

## Solutions: Module 3 MCQ Extension
16: Nested ignoring via `_`; 17: Must bind with name to inspect.

## Solutions: Module 4 Practical
Pair first component; Triple first > 0; Metrics focusing on cpu/errors; Node chain ignoring deeper; unnamed local variable discard.

## Solutions: Module 5 Errors
Guard using `_` invalid -> needs name; `_` not valid method/class identifier; cannot reference ignored component post-match.

## Solutions: Module 6 Techniques
Visitor refactor with `_`; signaled intentional disregard reduces cognitive load.

## Solutions: Module 7 Exercise
Deep record pattern: only top-level components bound; inner components replaced with `_`.

---

### End Notes
Unnamed patterns & variables refine Java pattern matching ergonomics, encouraging cleaner, intention-revealing code. Apply them judiciously to highlight relevant data while minimizing distraction.
