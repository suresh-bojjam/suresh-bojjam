# String Templates (Java 21) – Comprehensive Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

Java 21 (Preview) introduces String Templates (JEP 430/439 lineage) providing a safer, more expressive way to compose strings, integrating template processors with embedded expressions for formatting, escaping, SQL safety, and internationalization.

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
String concatenation and format operations can be verbose, error-prone, and risky (e.g., SQL injection). String Templates unify interpolation with pluggable processors that can validate and transform embedded expressions safely.

### 1.2 Core Concepts
| Concept | Description |
|---------|-------------|
| Template Literal | A quoted string with embedded expressions `${...}` |
| Processor | A functional interface transforming a template into final output |
| Embedded Expression | Java expression inside `${ }` evaluated and passed to processor |
| Raw String | Literal content plus expression results optionally sanitized |
| Formatter Processor | Built-in processor for format-style output |

### 1.3 Basic Example (Preview Syntax)
```java
import static java.lang.StringTemplate.RAW; // preview

int a = 3, b = 4;
StringTemplate st = RAW."Sum: ${a + b}"; // creates a template instance
String result = st.interpolate(); // default raw interpolation -> "Sum: 7"
```

### 1.4 Using a Processor
```java
import static java.lang.StringTemplate.RAW;
import static java.lang.StringTemplate.STR;

String name = "Ada";
String greeting = STR."Hello, ${name}!"; // STR is a processor -> returns String directly
```

### 1.5 ASCII Architecture Diagram
```
+----------------------------+
| Template Literal Source    |
+---------------+------------+
                | parse & evaluate
                v
        +------------------+            +---------------------+
        | StringTemplate   |  ------->  | Processor (STR, custom) |
        +--------+---------+            +-----------+---------+
                 | expressions + fragments            |
                 v                                     v
           Final Interpolated String            Validation / Escaping
```

### 1.6 Processors
- `RAW` -> yields `StringTemplate` object; later call `.interpolate()`.
- `STR` -> immediate basic interpolation -> String.
- Custom processors implement transformation (e.g., SQL parameterization, JSON escaping).

### 1.7 Embedded Expressions Semantics
Compile-time checks: expressions are typed; no implicit toString errors—each segment converted via standard string conversion (similar to `String.valueOf`).

### 1.8 Safety Advantages
Custom processors can reject unsafe patterns (e.g., embedded raw SQL keywords outside placeholders) reducing injection risks.

### 1.9 Exercise Set (Basics)
1. Create a RAW template and interpolate manually.
2. Use STR to interpolate name and age.
3. Show arithmetic inside `${}` producing computed value.
4. Create a custom processor that uppercases expressions.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Template Structure
A template has: fragments (string parts) + values (evaluated expressions). Processor receives both.

### 2.2 Building a Custom Processor
```java
import java.lang.invoke.MethodHandles;
import java.lang.StringTemplate;
import java.util.stream.Collectors;

StringTemplate.Processor<String, RuntimeException> UPPER = st -> {
    return st.fragments().get(0) + st.values().stream()
        .map(v -> v.toString().toUpperCase())
        .collect(Collectors.joining());
};

String out = UPPER."Values: ${"one"} ${"two"}"; // Values: ONE TWO
```

### 2.3 Validation Example (SQL-like)
```java
StringTemplate.Processor<String, IllegalArgumentException> SAFE_SQL = st -> {
    // naive example: forbid 'DROP'
    for (Object v : st.values()) {
        if (v instanceof String s && s.toUpperCase().contains("DROP")) {
            throw new IllegalArgumentException("Dangerous token");
        }
    }
    // simple join
    StringBuilder sb = new StringBuilder();
    for (int i=0;i<st.fragments().size();i++) {
        sb.append(st.fragments().get(i));
        if (i < st.values().size()) sb.append(st.values().get(i));
    }
    return sb.toString();
};

String query = SAFE_SQL."SELECT * FROM users WHERE name = ${userName}";
```

### 2.4 Internationalization Processor Concept
Processor could map placeholders to localized messages or format numbers with locale-specific patterns.

### 2.5 Nesting & Composition
Templates can include results of other template interpolations inside expressions.

### 2.6 Performance Considerations
- Minimizes temporary StringBuilder overhead vs naive concatenation.
- Processors can batch operations on values.

### 2.7 Security Patterns
Whitelist allowed characters or escape sequences for web output (HTML escaping) within processor.

### 2.8 Edge Cases
- Empty expressions -> valid if produce empty string.
- Null values -> converted to "null".
- Multiple consecutive expressions -> fragments alignment ensures deterministic concatenation.

### 2.9 Processor Error Handling
Throw checked or unchecked exceptions; if processor declares checked type it propagates; caller must handle.

### 2.10 Migration from `String.format`
Replace `%s` placeholders with `${expr}`; processors can replicate formatting rules.

### 2.11 Exercise Set (In-Depth)
1. Write a processor that JSON-escapes values.
2. Add validation to reject values exceeding length 50.
3. Implement numeric formatting (two decimals) in expressions.
4. Compose nested template within expression.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Primary purpose of String Templates? | Faster class loading | Safer & richer string composition | Binary serialization | Memory management | B | Focus on safety & expressiveness. |
| 2 | RAW processor returns? | String directly | StringTemplate object | int length | List of values | B | RAW yields template for later interpolation. |
| 3 | STR processor effect? | Returns template | Throws exception | Direct interpolation to String | Escapes JSON automatically | C | STR yields final String. |
| 4 | Embedded expression syntax? | $(expr) | ${expr} | #[expr] | <<expr>> | B | `${}` is correct. |
| 5 | Custom processor implements? | ClassLoader | `StringTemplate.Processor` | Runnable | Supplier | B | Functional interface. |
| 6 | Null value becomes? | Empty | null | "null" | Exception | C | Standard string conversion. |
| 7 | Processor can? | Alter fragments & values | Only read values | Only read fragments | Modify bytecode | A | It can transform content. |
| 8 | Security use-case? | Replace GC | Prevent injection | Change JVM args | Disable JIT | B | Validate/escape. |
| 9 | Which replaces String.format? | `%d` placeholders | `${}` with processors | XML tags | Reflection | B | Use `${}` syntax. |
| 10 | Nesting templates allowed? | No | Only top-level | Yes in expressions | Only if flagged | C | Expressions can contain nested results. |
| 11 | Handling processor error? | Ignored | Propagated exception | Silently logged | Always wrapped | B | Exceptions propagate. |
| 12 | Performance benefit? | Avoids CPU usage | Reduces intermediate concatenations | Disables GC | Increases network speed | B | Efficient composition. |
| 13 | Fragments represent? | Expressions | Literal parts between expressions | Entire final string | Processor state | B | They are raw string pieces. |
| 14 | Expressions typed when? | Runtime only | Compile-time type checking | After GC | During JIT only | B | Compile-time typed. |
| 15 | Unsafe pattern detection location? | JVM | Processor code | Class loader | JIT | B | Custom processor logic.

### MCQ Extension
16. Replacing nulls with custom token handled by? (A: custom processor)  
17. Limiting value length enforced where? (A: processor validation)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Basic STR Interpolation
```java
import static java.lang.StringTemplate.STR;
String user = "Lin";
String msg = STR."Welcome ${user}!";
```

### 4.2 RAW then Manual Interpolate
```java
import static java.lang.StringTemplate.RAW;
StringTemplate tmpl = RAW."Pi approx ${Math.PI}";
String out = tmpl.interpolate();
```

### 4.3 Uppercase Processor
```java
StringTemplate.Processor<String, RuntimeException> UPPER = st -> {
    StringBuilder sb = new StringBuilder();
    for (int i=0;i<st.fragments().size();i++) {
        sb.append(st.fragments().get(i));
        if (i < st.values().size()) sb.append(st.values().get(i).toString().toUpperCase());
    }
    return sb.toString();
};
String up = UPPER."Names: ${"alice"} ${"bob"}"; // Names: ALICE BOB
```

### 4.4 JSON Escaping Processor (Simplified)
```java
StringTemplate.Processor<String, RuntimeException> JSON = st -> {
    String esc = st.values().stream()
        .map(v -> v.toString().replace("\"", "\\\"").replace("\n", "\\n"))
        .collect(java.util.stream.Collectors.joining(","));
    return "[" + esc + "]";
};
String json = JSON."${"A"} ${"B\nC"}"; // [A,B\nC]
```

### 4.5 Safe SQL Processor
```java
StringTemplate.Processor<String, IllegalArgumentException> SAFE_SQL = st -> {
    for (Object v : st.values()) {
        String s = v.toString().toUpperCase();
        if (s.contains("DROP") || s.contains("TRUNCATE")) throw new IllegalArgumentException("Forbidden token");
    }
    StringBuilder sb = new StringBuilder();
    for (int i=0;i<st.fragments().size();i++) {
        sb.append(st.fragments().get(i));
        if (i < st.values().size()) sb.append(st.values().get(i));
    }
    return sb.toString();
};
String q = SAFE_SQL."SELECT * FROM users WHERE name=${name}";
```

### 4.6 Length Validation Processor
```java
StringTemplate.Processor<String, IllegalArgumentException> LIMIT = st -> {
    for (Object v : st.values()) if (v.toString().length() > 50) throw new IllegalArgumentException("Value too long");
    return st.interpolate();
};
```

### 4.7 Nested Template
```java
String inner = STR."Inner ${10 * 2}"; // "Inner 20"
String outer = STR."Outer sees: ${inner}"; // "Outer sees: Inner 20"
```

### 4.8 Exercise Set (Practical)
1. Implement processor replacing null with "(none)".
2. Add processor performing HTML escaping.
3. Compose nested templates for greeting and time.
4. Create a validator rejecting numbers < 0.
5. Build a format-like processor adding brackets around each value.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Missing Preview Feature Flag
Using templates without enabling preview -> compilation error requiring `--enable-preview`.

### 5.2 Wrong Processor Type
Returning wrong generic type leads to compilation error or unchecked cast warnings.

### 5.3 NullPointerException in Processor
Dereferencing `v.toString()` when `v` is null (unless handled).

### 5.4 IllegalArgumentException from Validation
Thrown by custom processor; must be caught if program flow expects fallback.

### 5.5 Syntax Error
Mistyped `${ expr }` -> e.g., `$[expr]` invalid -> compile-time failure.

### 5.6 Exercise Set (Errors)
Identify and fix: preview flag missing, null handling oversight, incorrect fragment assembly order.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Central Processor Registry
Keep processors in a utility class for reuse & versioning.

### 6.2 Defensive Escaping Layers
Chain processors or have one that sanitizes then formats.

### 6.3 Performance Profiling
Benchmark STR vs `String.format` for simple concatenations.

### 6.4 Hybrid Formatting
Insert pre-formatted numbers (e.g., `DecimalFormat`) inside expressions.

### 6.5 Logging Context Injection
Processor appends trace-id for log correlation.

### 6.6 Fallback Strategy
Wrap invocation in try-catch, fallback to RAW interpolation when custom processor fails.

### 6.7 Testing Processors
Unit test fragments & values separately; property-based testing for escaping.

### 6.8 Security Hardening
Reject control chars, enforce length limits, whitelist patterns before interpolation.

### 6.9 Extending to i18n
Processor selects locale-specific formatting for numbers/dates.

### 6.10 Exercise Set (Techniques)
Implement chain: validation -> escaping -> formatting; measure overhead.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. Advantages over concatenation or String.format?
2. Role of RAW vs STR processors.
3. How custom processors improve security.
4. Handling null expressions elegantly.
5. Preview feature activation steps.
6. Performance considerations.
7. Interpolation life cycle details.
8. Potential pitfalls in validation logic.
9. Migration strategy from existing formatting utilities.
10. Extending templates for localization.

### Exercise
Design a secure email templating system with placeholders and HTML escaping using processors.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
RAW creation + interpolate; STR with name & age; arithmetic example; uppercase processor sample.

## Solutions: Module 2 Exercises
JSON escape processor; length validation; numeric formatting using `String.format("%.2f", value)` inside expression; nested template example.

## Solutions: Module 3 MCQ Extension
16: Custom processor modifies null handling; 17: Processor validation enforces length.

## Solutions: Module 4 Practical
Null replacement processor; HTML escape using character mapping; nested greeting + time; validator rejecting negative numbers; bracket format processor.

## Solutions: Module 5 Errors
Compile with `--enable-preview --release 21`; add null guard before toString; correct fragment loop ordering.

## Solutions: Module 6 Techniques
Registry pattern centralizes processors; chain demonstration; profiling approach.

## Solutions: Module 7 Exercise
Secure email template: validation processor verifying allowed tags; escape processor converting `<>&"'`; formatting processor assembling final HTML.

---

### End Notes
String Templates empower safer, clearer interpolation and open a pathway for domain-specific processors (security, i18n, logging). Adopt gradually: start with STR, introduce custom processors for critical paths, enforce validation & escaping consistently.
