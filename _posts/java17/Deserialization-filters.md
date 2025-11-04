# Java Deserialization Filters (Java 9+ through Java 21) – In-Depth Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

NOTE: Deserialization filters originated (JEP 290) in Java 9; Java 21 context reinforces secure baseline practices. This guide covers configuring global & per-stream filters, patterns, and defensive strategies.

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
Java native serialization historically susceptible to remote code execution (gadget chains) and resource exhaustion. Deserialization filters provide a programmatic and configurable way to whitelist/blacklist classes, limit graph size, and reject dangerous inputs early.

### 1.2 Core Concepts
- Global Filter: Applied once set on `ObjectInputFilter.Config.setSerialFilter(...)` affects all subsequently created `ObjectInputStream`s without explicit per-stream filter.
- Per-Stream Filter: Attached to individual `ObjectInputStream` via `setObjectInputFilter` for granular control.
- Evaluation Result: `ALLOWED`, `REJECTED`, `UNDECIDED` -> subsequent filters continue until final decision.

### 1.3 ObjectInputFilter API Overview
```java
ObjectInputFilter filter = info -> {
    Class<?> cl = info.serialClass();
    if (cl != null) {
        if (cl.getName().startsWith("java.util.")) return ObjectInputFilter.Status.ALLOWED;
        if (cl.getName().equals("java.lang.Runtime")) return ObjectInputFilter.Status.REJECTED;
    }
    if (info.references() > 10_000) return ObjectInputFilter.Status.REJECTED;
    return ObjectInputFilter.Status.UNDECIDED; // delegate to next/global
};
```

### 1.4 ASCII Flow Diagram
```
+------------------+     +--------------------+     +-------------------+
| Incoming Stream  | --> | Per-Stream Filter  | --> | Global Filter     |
+------------------+     +---------+----------+     +---------+---------+
                                  |                        |
                              Decision                 Final Decision
```
(Actual evaluation order: per-stream first, then global if UNDECIDED.)

### 1.5 Filter Grammar (Configuration String)
System property / command-line `-Djava.io.serialization.serialFilter` uses grammar:
```
pattern := ("maxdepth=" number) | ("maxrefs=" number) | ("maxbytes=" number) | ("maxarray=" number)
         | class pattern (accept/reject) rules
```
Examples:
```
-Djava.io.serialization.serialFilter="maxdepth=20;maxrefs=500;java.base/*;!com.bad.gadget.*"
```

### 1.6 Basic Example
```java
try (ObjectInputStream ois = new ObjectInputStream(in)) {
    ois.setObjectInputFilter(filter); // per-stream
    Object obj = ois.readObject();
}
```

### 1.7 Exercise Set (Basics)
1. Create a filter that only allows classes under `com.example.model` and rejects others.
2. Configure a global filter limiting `maxdepth=10` and rejecting package `com.malware.*`.
3. Demonstrate per-stream override of global filter.
4. Reject if references exceed 1000.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Filter Evaluation Order
Per-stream filter evaluated first; if `UNDECIDED`, global filter then evaluates. If both UNDECIDED, default allow continues (unless security manager / future constraints). Always write explicit rules; rely less on implicit allow.

### 2.2 Limits
| Limit | Purpose |
|-------|---------|
| `maxdepth` | Prevent excessively deep object graphs (stack exhaustion) |
| `maxrefs` | Prevent huge graphs (heap exhaustion) |
| `maxbytes` | Restrict total number of bytes read for object data |
| `maxarray` | Limit single array length |

### 2.3 Class Pattern Matching
Patterns like `java.base/java.util.*` or `!com.evil.*` with leading `!` to reject. Module-based prefixes supported.

### 2.4 Combining Programmatic + Declarative Filters
Set a global filter via property, then refine in code for special streams (e.g., internal trusted channel vs external input).

### 2.5 Example: Layered Filtering
```java
ObjectInputFilter base = ObjectInputFilter.Config.getSerialFilter();
ObjectInputFilter tight = info -> {
    if (info.references() > 200) return ObjectInputFilter.Status.REJECTED;
    Class<?> cl = info.serialClass();
    if (cl != null && cl.getName().startsWith("com.example.secure")) return ObjectInputFilter.Status.ALLOWED;
    return ObjectInputFilter.Status.UNDECIDED;
};
ObjectInputFilter chained = ObjectInputFilter.merge(tight, base);
```

### 2.6 Global Filter Registration
```java
ObjectInputFilter.Config.setSerialFilter(info -> {
    if (info.references() > 5000) return ObjectInputFilter.Status.REJECTED;
    Class<?> cl = info.serialClass();
    if (cl != null && cl.getName().startsWith("java.lang.invoke.")) return ObjectInputFilter.Status.REJECTED;
    return ObjectInputFilter.Status.UNDECIDED;
});
```

### 2.7 Logging Decisions
Implement filter to log REJECT events for audit.

### 2.8 Performance Considerations
Filtering adds minimal overhead (simple checks per object). Provide O(1) decisions: avoid regex; prefer prefix checks.

### 2.9 Deserialization Alternatives
Consider JSON/CBOR using libraries for untrusted input; native serialization only for trusted boundaries.

### 2.10 Exercise Set (In-Depth)
1. Implement layered filter merging strict with global.
2. Add array length limit using programmatic filter.
3. Log and count rejected classes.
4. Provide fallback to allow `java.time.*` only and reject others.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Deserialization filters introduced by? | JEP 290 | JEP 431 | JEP 441 | JEP 378 | A | JEP 290 (Java 9). |
| 2 | Per-stream filter evaluation order? | After global | Before global | Random | Parallel | B | Stream filter first. |
| 3 | Status allowing next filter evaluation? | ALLOWED | REJECTED | UNDECIDED | ERROR | C | UNDECIDED passes to next. |
| 4 | Reject pattern prefix? | `-` | `!` | `#` | `~` | B | `!` indicates rejection. |
| 5 | Purpose of `maxdepth`? | Limit array length | Limit object graph nesting | Limit bytes | Limit classes loaded | B | Depth constraint. |
| 6 | `ObjectInputFilter.merge(a,b)` semantics? | a then b if UNDECIDED | b then a always | Only b used | Random order | A | Per docs merges evaluation order. |
| 7 | Filtering overhead typically? | High | Negligible | Blocks all threads | Requires reflection every object | B | Simple checks. |
| 8 | Global filter config property? | `java.io.globalFilter` | `java.io.serialization.serialFilter` | `serialization.filter` | `java.serialization.filter` | B | Correct property name. |
| 9 | UNDECIDED + no other filters -> result? | Rejected | Allowed | Error | Logged only | B | Allowed continuation. |
| 10 | Good strategy for performance? | Heavy regex each object | Prefix checks & counters | Deserialize then filter | Sleep between checks | B | Keep O(1). |
| 11 | Danger of not limiting `maxrefs`? | Performance improvement | Potential memory exhaustion | Faster GC | Safer code | B | Large graphs -> memory issues. |
| 12 | To disallow everything except one package? | Allow all then reject | Start with reject all pattern | Use whitelist for allowed package + reject wildcard | Use default only | C | Combine allow + reject. |
| 13 | Filtering untrusted input recommended? | Yes | No | Only if encrypted | Only on Unix | A | Essential for security. |
| 14 | `REJECTED` status effect? | Continues to next filter | Immediately aborts deserialization | Logs only | Converts to UNDECIDED | B | Stops process. |
| 15 | Which limit stops huge single array? | `maxarray` | `maxrefs` | `maxdepth` | `maxbytes` | A | Array length constraint.

### MCQ Extension Exercise
Add 2 MCQs: one about `maxbytes`, one about logging rejected classes.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Whitelist Filter
Programmatic filter allowing only domain model classes.

### Exercise 4.2: Global + Per-Stream Override
Set global filter then override on a trusted internal socket.

### Exercise 4.3: Array Length Limit
Reject arrays > 10_000 elements.

### Exercise 4.4: Reference Count Limit
Stop deserialization beyond 1000 references.

### Exercise 4.5: Logging Filter
Implement filter counting REJECTED attempts.

### Exercise 4.6: Merging Filters
Combine strict & lenient filters using `merge`.

### Exercise 4.7: Configuration String Usage
Start JVM with property to limit depth and bytes.

### Exercise 4.8: Fallback Default Strategy
Reject unknown packages except whitelisted ones.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Wrong Property Name
Misspelling system property results in filter not applied silently.

### 5.2 Null Filter Merge
Passing null to merge leads to NPE—guard against.

### 5.3 Late Filter Set
Setting global filter after streams created -> those streams not covered.

### 5.4 Pattern Syntax Mistake
Using spaces incorrectly or missing semicolons leads to invalid pattern parse (ignored or error in logs).

### 5.5 Exercise: Identify Errors
```java
ObjectInputFilter.Config.setSerialFilter(null); // 1
ObjectInputFilter f = ObjectInputFilter.merge(null, info -> ObjectInputFilter.Status.ALLOWED); // 2
System.setProperty("java.io.serialization.serialFilters", "maxdepth=5"); // 3 (typo)
```
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Defense-in-Depth
Combine filter + external format validation + least-privilege classpath.

### 6.2 Opportunistic Early Abort
Check reference count early to abort huge graphs quickly.

### 6.3 Layered Trust Zones
Different filters for internal vs external network input.

### 6.4 Audit Trail
Log class name & timestamp for REJECTED events.

### 6.5 Adaptive Limits
Increase limits for trusted sessions after handshake.

### 6.6 Fallback to Safer Formats
Use filters as safety net while migrating off Java serialization.

### 6.7 Monitoring Metrics
Expose counters (rejections, allowed objects) via JMX.

### 6.8 Testing Strategy
Inject malicious payload fixtures to verify rejection logic.

### 6.9 Performance Profiling
Benchmark filter overhead under high throughput using synthetic graphs.

### 6.10 Exercise: Implement metrics wrapper around filter counting each status.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Why implement deserialization filters?  
Reduce attack surface, mitigate gadget exploitation & resource exhaustion.
2. Difference between global & per-stream filter?  
Global applies across streams; per-stream allows contextual overrides.
3. Meaning of UNDECIDED status?  
Delegates decision to next filter; eventual allowed if no rejection.
4. How to reject large object graphs?  
Use `maxdepth`, `maxrefs`, appropriate pattern rules.
5. Strategy to phase out native serialization?  
Introduce filters + migrate to safer formats (JSON, protobuf) gradually.
6. Logging best practices?  
Log class, reason, timestamp; avoid sensitive object data.
7. Performance impact?  
Minimal; constant-time checks recommended.
8. How to allow only whitelisted packages?  
Allow patterns for desired namespace; reject wildcard for others.
9. Why merge filters?  
Compose layered policies (e.g., baseline + contextual restrictions).
10. Handling unknown classes?  
Reject or default to safe placeholder; never instantiate untrusted unknown types.

### Exercise: Explain role of `maxbytes` vs `maxrefs`.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
Whitelist filter, global property usage, per-stream override, references limit snippet.

## Solutions: Module 2 Exercises
Layered merge, array length limit, logging sample, fallback strategy.

## Solutions: Module 3 MCQ Extension
`maxbytes` caps total bytes read for object data; logging MCQ identifies using audit filter.

## Solutions: Module 5 Error Identification (5.5)
1. Setting serial filter null -> disables protection (avoid). 2. merge(null, ...) leads NPE. 3. Wrong property key `serialFilters` ineffective (should be `serialFilter`).

## Solutions: Module 6 Exercise
Metrics wrapper increments counters based on status returned.

## Solutions: Module 7 Exercise
`maxbytes` limits encoded size; `maxrefs` counts object handles in graph—both mitigate resource exhaustion differently.

---

### End Notes
Deserialization filters are critical safeguards: implement strict whitelists, resource limits, and logging to reduce risk while planning migration away from native serialization where possible.
