# Unnamed Classes and Instance Main Methods (Java 21) – Comprehensive Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

Unnamed classes and instance main methods (Preview, JEP 445 lineage) reduce ceremony for small programs, scripts, and educational examples. They enable writing Java source with top-level statements or an instance `main` method without declaring a public class explicitly.

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
Traditional Java requires boilerplate: `public class X { public static void main(String[] args) { ... } }`. For quick scripts or teaching, this verbosity hinders adoption. Unnamed classes allow source files comprised of statements and declarations executed as a program, streamlining authoring.

### 1.2 Core Concepts
| Concept | Description |
|---------|-------------|
| Unnamed Class | Implicit class wrapper around top-level statements and declarations |
| Instance Main Method | `void main()` or `void main(String... args)` instance method executed instead of static main |
| Launch Protocol | JVM recognizes preview feature and runs implicit main sequence |
| Simplicity | Reduces need for public class and static context |

### 1.3 Basic Unnamed Program Example
`Hello.java` contents:
```java
System.out.println("Hello, World!");
```
Compile & run with preview:
```bash
javac --enable-preview --release 21 Hello.java
java  --enable-preview Hello
```

### 1.4 Instance Main Method Example
```java
void main() {
    System.out.println("Instance main invoked");
}
```
No class declaration; compiler wraps in unnamed class; runtime creates instance and calls `main`.

### 1.5 ASCII Diagram – Execution Flow
```
+------------------+      +-------------------------+
| Source File      | ---> | Implicit Unnamed Class  |
| (top statements) |      | + instance main method  |
+---------+--------+      +-----------+-------------+
                                  |
                                  v
                           Program Execution
```

### 1.6 Command-line Arguments
```java
void main(String... args) {
    System.out.println("Args count=" + args.length);
}
```

### 1.7 Allowed Top-Level Constructs
- Statements (executed before instance main? Depends on JEP specification ordering).
- Import declarations.
- Method declarations (instance). 
- Field declarations (instance fields of unnamed class).

### 1.8 Restrictions
- No explicit package declaration for unnamed class usage (script style).
- Preview feature flags required.
- Not suitable for large applications or library distribution.

### 1.9 Exercise Set (Basics)
1. Create a one-line unnamed program printing arguments length.
2. Add instance field and print inside `main`.
3. Use import and call a standard library method.
4. Add a helper instance method and invoke from `main`.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Execution Order
Top-level statements execute in textual order prior to or as part of instance initialization (preview design ensures intuitive sequential execution). Then `main` instance method invoked.

### 2.2 Differences vs Static Main
Instance main can access instance fields directly without `static` qualifiers; encourages simpler patterns for small scripts.

### 2.3 Interoperability with Traditional Classes
Unnamed class cannot be referenced by name externally; for expansion, migrate to named public class.

### 2.4 Transition Path
Refactor unnamed program to named class:
```java
class App { void main(String... args) { /* same body */ } }
```
Add static main if distributing:
```java
public class App { public static void main(String[] a){ new App().main(a); } }
```

### 2.5 Fields & State
```java
int counter = 0;
void main() { counter++; System.out.println(counter); }
```
State preserved for duration of single execution.

### 2.6 Helper Methods
```java
String shout(String s){ return s.toUpperCase(); }
void main(){ System.out.println(shout("loom")); }
```

### 2.7 Error Handling
Use try/catch normally:
```java
void main(){
    try { Files.readString(Path.of("missing.txt")); } catch (IOException e){ System.out.println("missing"); }
}
```

### 2.8 Imports Usage
```java
import java.time.*;
void main(){ System.out.println(Instant.now()); }
```

### 2.9 Command-line Parameters Parsing
```java
void main(String... args){
    if (args.length == 0) System.out.println("No args");
}
```

### 2.10 Limitations & Migration Considerations
Refactoring required to add packaging, unit tests integration, modularization.

### 2.11 Exercise Set (In-Depth)
1. Demonstrate field initialization before main execution.
2. Implement helper parsing method called from main.
3. Convert unnamed program to named class preserving functionality.
4. Add static main wrapper for distribution.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Primary goal of unnamed classes? | Enhance GC | Reduce boilerplate for small programs | Improve JVM bytecode verification | Replace packages | B | Focus on simplicity. |
| 2 | Instance main differs from static main by? | Running on separate JVM | Being an instance method | Not allowed to use args | Auto concurrency | B | Instance vs static. |
| 3 | Preview feature flag needed? | No | Yes | Only for static main | Only on Linux | B | Must enable preview. |
| 4 | Top-level statement placement indicates? | Class fields | Executable code | Imports only | Annotations only | B | Direct executable statements. |
| 5 | Migration to distributable form requires? | Delete code | Add public class & static main | Use C++ wrapper | Remove imports | B | Introduce named class. |
| 6 | Instance fields accessed? | Through class loader | Directly within instance main | Only via getters | Not allowed | B | Instance context. |
| 7 | Unnamed class referenced from another file? | Yes by name | No direct reference | Only via reflection | Only via package-private | B | Not named. |
| 8 | Command-line args signature? | void main(String... args) | void main(int) | main(List<String>) | main() only | A | Varargs accepted. |
| 9 | Purpose of helper methods? | Provide extra logic reused from main | Replace imports | Guarantee static initialization | Avoid exceptions | A | Reusable logic. |
| 10 | Error handling inside unnamed program? | Not allowed | Normal try/catch | Only static catch | Requires wrapper | B | Same semantics. |
| 11 | Adding static main for distribution does? | Break program | Provide conventional entry point | Disable instance main | Remove preview requirement | B | Adds standard entry. |
| 12 | Unnamed programs ideal for? | Large enterprise apps | Quick scripts & teaching | Bytecode engineering | JVM tuning | B | Lightweight usage. |
| 13 | Field initialization occurs? | After main | Before main executes | Randomly | Not supported | B | Standard order prior to method logic. |
| 14 | Without preview flag compile? | Succeeds silently | Fails with messaging | Produces warning only | Always runs | B | Preview required. |
| 15 | Imports allowed? | Yes | No | Only java.lang | Only custom modules | A | Standard import usage.

### MCQ Extension
16. To evolve unnamed program into library you first? (A: Create named public class)  
17. Accessing state across methods relies on? (A: Instance fields of implicit class)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Minimal Script
```java
System.out.println("Hi!");
```

### 4.2 Instance Field & Main
```java
int count = 0;
void main(){ count++; System.out.println(count); }
```

### 4.3 Helper Method Use
```java
int doubleIt(int x){ return x*2; }
void main(){ System.out.println(doubleIt(5)); }
```

### 4.4 Args Handling
```java
void main(String... args){ System.out.println("args=" + args.length); }
```

### 4.5 Migration to Named Class
Unnamed:
```java
void main(){ System.out.println("Hello"); }
```
Named:
```java
public class HelloApp { public static void main(String[] a){ new HelloApp().main(a); } void main(String... a){ System.out.println("Hello"); }}
```

### 4.6 Error Handling Example
```java
import java.nio.file.*;
void main(){
    try { Files.readString(Path.of("missing")); } catch (Exception e){ System.out.println("missing file"); }
}
```

### 4.7 Field Initialization Order
```java
String msg = init();
String init(){ return "Ready"; }
void main(){ System.out.println(msg); }
```

### 4.8 Exercise Set (Practical)
1. Add counter field and increment each run.
2. Use helper to format string.
3. Add args parsing for `--name`.
4. Migrate to named class with static main.
5. Implement simple file read with error handling.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Missing Preview Flag
Compiling without `--enable-preview` -> compilation error referencing preview features.

### 5.2 Duplicate `main` Signatures
Having both instance and static main in unnamed context leads to ambiguity (avoid mixing).

### 5.3 Package Declaration Misuse
Adding `package` for unnamed script may cause restrictions or shift semantics; keep default if using unnamed class style.

### 5.4 Unreferenced Imports
Unused imports still compile but may trigger warnings; not an error yet.

### 5.5 Incorrect Args Signature
Using `int main(String[])` invalid; must be `void` return.

### 5.6 Exercise Set (Errors)
Identify: missing preview flag, wrong main signature, mixing static & instance main unnecessarily.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Rapid Prototyping
Leverage unnamed scripts for quick algorithm trials before formalizing.

### 6.2 Educational Onboarding
Teach fundamentals without upfront class ceremony.

### 6.3 Gradual Expansion
Start unnamed, then migrate to named class as complexity grows.

### 6.4 Instance State Use
Simplify small tools with instance fields instead of static variables.

### 6.5 CLI Parsing Helper
Add small parsing helpers; later refactor to libraries.

### 6.6 Testing Approach
Wrap code into named class for test harness integration; maintain script version for examples.

### 6.7 Tooling Integration
Editor snippets for unnamed program templates.

### 6.8 Error Logging Strategy
Inline minimal logging; expand to structured logging post-migration.

### 6.9 Performance Micro-Bench Drafts
Quick micro-bench scaffolds before migrating to JMH formal harness.

### 6.10 Exercise Set (Techniques)
Design path from single-file script to modular project with minimal friction.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. Describe unnamed classes feature purpose.
2. Compare instance main vs traditional static main.
3. How to migrate unnamed script to production-ready class?
4. Discuss limitations of unnamed programs.
5. Benefits for education.
6. Potential pitfalls if overused.
7. Handling arguments in unnamed program.
8. Managing state across helper methods.
9. Why preview flags required?
10. Expansion strategy for scaling.

### Exercise
Design a simple CLI todo manager starting as unnamed script, then outline migration steps.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
One-line program; instance field usage; import demonstration; helper invocation.

## Solutions: Module 2 Exercises
Field initialization before main; helper parse method; conversion to named class; static wrapper addition.

## Solutions: Module 3 MCQ Extension
16: Create named public class; 17: Instance fields hold state.

## Solutions: Module 4 Practical
Counter increment; formatting helper; args parse for --name; migration snippet; file read with try/catch.

## Solutions: Module 5 Errors
Add preview flag; correct `void main`; avoid static+instance duplication.

## Solutions: Module 6 Techniques
Roadmap from script to modular: add package, convert to public class, introduce build tool & tests.

## Solutions: Module 7 Exercise
Todo manager: start with simple list operations; migrate to named class; add persistence & argument parsing enhancements.

---

### End Notes
Unnamed classes & instance main methods lower the barrier to entry for Java scripting and pedagogy. Use them for rapid prototyping and teaching; migrate to standard class structures as complexity increases.
