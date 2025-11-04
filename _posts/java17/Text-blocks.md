# Java Text Blocks (Java 15+ Final in 17) – In-Depth Training

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
Text blocks simplify writing multi-line string literals with predictable indentation, fewer escape sequences, and improved readability for embedded data (JSON, SQL, HTML, XML).

### 1.2 Syntax Overview
A text block is delimited by three double quotes `"""` start and end. Example:
```java
String json = """
{
  "name": "Alice",
  "age": 30
}
""";
```

### 1.3 Indentation & Stripping Rules
- Common leading whitespace on all non-blank lines removed.
- First newline after opening `"""` is ignored (optional for layout).
- Trailing spaces preserved.

ASCII Visualization:
```
Source (with leading indentation):
    """
        Line A
        Line B
    """
Processed -> "Line A\nLine B\n"
```

### 1.4 Escape Sequence Reduction
Need fewer escapes: `"` rarely requires escaping, no need for `\n` for each line. Still can use escapes for special characters (e.g., `\t`, Unicode).

### 1.5 Embedded Expressions (Not Supported Natively)
Text blocks are plain strings; use `String.format`, concatenation, or templating frameworks.
```java
String name = "Bob";
String html = """
<html><body><h1>Hello %s</h1></body></html>
""".formatted(name);
```

### 1.6 Comparison with Traditional Strings
| Aspect | Traditional Multi-line | Text Block |
|--------|------------------------|------------|
| Escapes | Many `\n` required | Newlines literal |
| Indent control | Manual | Automatic common indent removal |
| Readability | Lower for structured data | Higher |
| Trailing newline | Must add explicitly | Often implicit |

### 1.7 Example: SQL Query
```java
String sql = """
SELECT id, name, created_at
FROM users
WHERE status = 'ACTIVE'
ORDER BY created_at DESC
LIMIT 10
""";
```

### 1.8 Exercise Set (Basics)
1. Create a JSON text block representing a user profile.
2. Write a multi-line HTML snippet as a text block.
3. Format a text block with placeholders.
4. Show difference between text block indentation vs manual string concatenation.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Indentation Algorithm Details
The common indent is the minimum leading whitespace across all non-blank lines excluding the first line if it is blank. That indent is stripped. Additional indentation kept relative to baseline.

### 2.2 Control Over Trailing Newlines
Text block retains a final newline unless ending delimiter placed on same line as content.
```java
String s1 = """
Hello
"""; // contains "Hello\n"
String s2 = """Hello"""; // contains "Hello"
```

### 2.3 Combining with `String#stripIndent` & `translateEscapes`
`stripIndent()` further normalizes indentation; `translateEscapes()` processes escape sequences inside text block at runtime.
```java
String raw = """
  A\tB\nC
""";
String translated = raw.translateEscapes();
```

### 2.4 Using `formatted()` and `String.format`
```java
String template = """
User: %s
Role: %s
""";
String out = template.formatted("alice", "ADMIN");
```

### 2.5 Escaping Triple Quotes
Include `"""` inside by escaping one quote:
```java
String meta = """
He said: \"""Hello\"""
""";
```

### 2.6 Performance Considerations
Compile-time constant if no runtime concatenation; similar footprint to traditional strings. Use for readability rather than micro-performance gains.

### 2.7 Integration with JSON / XML Builders
Text blocks help maintain templates; ensure placeholders sanitized to avoid injection.

### 2.8 Logging Multiline Messages
Maintain formatting clarity; avoid `\n` clutter.

### 2.9 Testing & Golden Files
Embed expected outputs as text blocks in test code for snapshot comparisons.

### 2.10 Exercise Set (In-Depth)
1. Create a template with `%s` placeholders and format with dynamic data.
2. Demonstrate difference when placing closing quotes on same line vs new line.
3. Use `translateEscapes()` to interpret `\t` and `\n` sequences.
4. Construct an XML configuration using a text block and parse it.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Text blocks delimiters? | `'''` | `"""` | `` ``` `` | `"` | B | Java uses triple double quotes. |
| 2 | First newline after opening delimiter? | Always kept | Always removed | Ignored if immediately after opener | Causes error | C | Ignored when directly after `"""`. |
| 3 | Indent removal based on? | Longest indent | Average indent | Minimum common indent | No removal | C | Common minimum removed. |
| 4 | Trailing newline behavior? | Never included | Always included | Depends on closing delimiter position | Random | C | Position determines inclusion. |
| 5 | Method to format placeholders? | `formatted()` | `place()` | `apply()` | `args()` | A | formatted available since Java 15. |
| 6 | Escape triple quotes inside block? | Impossible | Use `\"` | Use `\\"""` | Use backticks | B | Escape internal quotes. |
| 7 | Convert escapes runtime? | `translateEscapes()` | `runEscapes()` | `parseEscapes()` | `applyEscapes()` | A | Provided API. |
| 8 | Indent normalization helper? | `stripIndent()` | `normalizeIndent()` | `fixIndent()` | `trimIndent()` | A | Method exists. |
| 9 | Multi-line SQL improved by? | Removing escapes | Adding generics | Using annotations | Using reflection | A | Fewer escapes and clearer layout. |
| 10 | Content without trailing newline created by? | Adding space | Ending delimiter same line | Double closing quotes | Using `stripIndent()` | B | Closing on same line prevents newline. |
| 11 | Text block type? | New primitive | String | Char array | Template object | B | Produces java.lang.String. |
| 12 | `formatted()` returns? | New String instance | Modifies original | void | Optional<String> | A | Produces formatted string. |
| 13 | Use case for `translateEscapes()`? | At compile time | To interpret escape sequences occurring literally | To remove indent | To validate JSON | B | Converts escape sequences. |
| 14 | Relative indentation preservation? | Fully removed | Relative retained after baseline removal | Duplicated | Reversed | B | Only common baseline removed. |
| 15 | Text block adoption benefit? | Less readability | More boilerplate | Easier embedding of structured text | Slower execution | C | Improves clarity.

### MCQ Extension Exercise
Add 2 MCQs: one about trailing newline control, one about embedding placeholders.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: HTML Template
Create HTML snippet with dynamic title using `formatted()`.

### Exercise 4.2: SQL Query Parameterization
Prepare multi-line SQL (without user input interpolation) and display.

### Exercise 4.3: JSON Configuration
Generate config JSON text block and parse via library.

### Exercise 4.4: Translating Escapes
Use text block containing literal `\n` and apply `translateEscapes()`.

### Exercise 4.5: Indentation Experiment
Show effects of moving closing delimiter to same line.

### Exercise 4.6: Logging Multi-Line Error
Format stack trace portion in text block for logger.

### Exercise 4.7: Combining `stripIndent` & Trim
Demonstrate cleaning indentation and trimming ends.

### Exercise 4.8: Placeholder Formatting vs Concatenation
Compare readability and performance.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Unterminated Block
Missing closing `"""` leads to compilation error.

### 5.2 Illegal Escape
Incorrect sequence (e.g., `\q`) triggers compile-time error.

### 5.3 Misaligned Indent Expectation
Unexpected spaces remain when one line has fewer leading spaces than others; adjust alignment.

### 5.4 Placeholder Misuse
Using `%d` but passing String results in `IllegalFormatConversionException` at runtime.

### 5.5 `translateEscapes` Misunderstanding
Calling on block without escapes has no effect; expecting formatting change leads to confusion.

### 5.6 Exercise: Identify Errors
```java
String bad = """
Line1
  Line2
 Line3
""";
String fmt = """
Value: %d
""".formatted("text");
```

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Embedding DSLs
Use text blocks to embed small DSL scripts or config.

### 6.2 Indent Anchoring
Place opener on its own line with consistent indentation for predictable baseline removal.

### 6.3 Template Composition
Combine small blocks via concatenation or `StringBuilder` when dynamic sections large.

### 6.4 Reusable Templates Registry
Store text blocks in static constants for reuse.

### 6.5 Minimizing Escapes
Prefer actual characters; escape only necessary embedded quotes.

### 6.6 White-Space Control
Move closing delimiter to control trailing newline presence.

### 6.7 Diagnostic Output
Rapid prototyping of debug multi-line outputs.

### 6.8 Internationalization Prep
Use placeholders `%s` then apply localized strings.

### 6.9 Testing Snapshot Strings
Embed expected test outputs exactly as they should appear.

### 6.10 Exercise: Build a small registry of templates and format them with environment data.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Why text blocks instead of concatenated strings?  
Improved readability, reduced escaping, easier maintenance.
2. How is indentation determined?  
Minimum common leading whitespace among non-blank lines removed.
3. How to remove trailing newline?  
Place closing delimiter on same line as last content.
4. Are text blocks different type from String?  
No, they produce a `String`.
5. How format dynamic data?  
Use `formatted()` or `String.format()`.
6. When use `translateEscapes()`?  
To process escape sequences written literally in text block.
7. Performance impact vs traditional strings?  
Comparable; choose for readability.
8. How ensure no indentation stripping?  
Align lines so minimum indent is zero or add sentinel line.
9. Difference between `%s` placeholder and curly braces templating?  
`%s` uses format API; curly braces require custom template engine.
10. How to embed triple quotes inside?  
Escape internal quotes with backslash.

### Exercise: Explain difference between closing delimiter placement and resulting string content.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. JSON example.
2. HTML snippet using text block.
3. `formatted()` usage shown.
4. Indentation difference demonstration.

## Solutions: Module 2 Exercises
1. Template formatting code sample.
2. Closing delimiter comparison sample.
3. `translateEscapes` usage sample.
4. XML parse sample.

## Solutions: Module 3 MCQ Extension
1. Trailing newline: place closing delimiter same line to omit newline.
2. Placeholder formatting: use `%s` with `formatted()`.

## Solutions: Module 5 Error Identification (5.6)
Issues: Indentation irregular causing stray spaces; format using `%d` with String argument triggers `IllegalFormatConversionException`.

## Solutions: Module 6 Template Registry Exercise
Implement static map of keys to templates; apply formatted substitution.

## Solutions: Module 7 Exercise
Closing delimiter on separate line includes newline; same line excludes newline.

---

### End Notes
Use text blocks to elevate clarity for structured text, controlling indentation and trailing newlines deliberately while leveraging formatting APIs for dynamic content.
