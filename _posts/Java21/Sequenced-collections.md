# Java 21 Sequenced Collections – In-Depth Training (JEP 431)

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
Before Java 21, endpoint operations (first/last element) and reversed views were inconsistent across collections: `List` had index-based access, `Deque` had front/back operations, `LinkedHashMap` had ordered entries but no unified API. Sequenced Collections introduce uniform first/last access and live reversed views across lists, sets, and maps, improving API consistency and enabling clearer algorithms.
Introduced in Java 17 but standardized in Java 21.

### 1.2 New Interfaces
| Interface | Extends | Endpoint Ops | Reversed View | Description |
|-----------|--------|--------------|---------------|-------------|
| `SequencedCollection<E>` | `Collection<E>` | `getFirst()/getLast()` `addFirst()/addLast()` `removeFirst()/removeLast()` | `reversed()` | Ordered collection with double-ended operations |
| `SequencedSet<E>` | `Set<E>`, `SequencedCollection<E>` | Inherited | Inherited | Ordered set (iteration order) |
| `SequencedMap<K,V>` | `Map<K,V>` | `firstEntry()/lastEntry()` `pollFirstEntry()/pollLastEntry()` | `reversed()` | Ordered map with endpoint access |

### 1.3 Implementations in Java 21+
- `ArrayList` implements `SequencedCollection`.
- `LinkedHashSet` implements `SequencedSet`.
- `LinkedHashMap` implements `SequencedMap`.
(Other ordered collections may adopt sequenced interfaces over time.)

### 1.4 ASCII Hierarchy Diagram
```
                +-------------------------+
                |   SequencedCollection   |
                +------------+------------+
                             |
                +------------+------------------+
                |                               |
         (List implementations)          SequencedSet
                |                               |
           ArrayList                     LinkedHashSet

                +------------------+
                |  SequencedMap    |
                +--------+---------+
                         |
                     LinkedHashMap
```

### 1.5 Core Semantics
- Order preservation: respects encounter/insertion order of underlying implementation.
- Endpoint operations: O(1) on typical implementations (ArrayList, LinkedHashSet, LinkedHashMap).
- `reversed()` returns a LIVE view (modifications visible both directions).

### 1.6 Basic Examples
```java
SequencedCollection<String> sc = new ArrayList<>();
sc.addLast("B");
sc.addFirst("A"); // [A, B]
sc.addLast("C");  // [A, B, C]
System.out.println(sc.getFirst()); // A
System.out.println(sc.getLast());  // C
SequencedCollection<String> rev = sc.reversed();
System.out.println(rev); // [C, B, A]
rev.addFirst("Z");       // logically add at end of original
System.out.println(sc);  // [A, B, C, Z]
System.out.println(rev); // [Z, C, B, A]
```

### 1.7 SequencedSet Example
```java
SequencedSet<Integer> sset = new LinkedHashSet<>();
sset.addLast(10);
sset.addFirst(5);
sset.addLast(20); // [5,10,20]
int first = sset.getFirst(); // 5
int last = sset.getLast();   // 20
boolean moved = sset.addLast(10); // false, order unchanged
```

### 1.8 SequencedMap Example
```java
SequencedMap<Integer,String> smap = new LinkedHashMap<>();
smap.put(1,"one");
smap.put(2,"two");
smap.put(3,"three");
System.out.println(smap.firstEntry()); // 1=one
System.out.println(smap.lastEntry());  // 3=three
var revMap = smap.reversed();
System.out.println(revMap); // {3=three, 2=two, 1=one}
revMap.put(4,"four");
System.out.println(smap);   // {1=one,2=two,3=three,4=four}
```

### 1.9 Exercise Set (Basics)
1. Build a `SequencedCollection` of strings and perform all endpoint ops.
2. Demonstrate live reversed view updates.
3. Show uniqueness behavior in `SequencedSet` when re-adding existing element.
4. Use `SequencedMap` to access and remove endpoints.

(See Module 8 for solutions.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Live Reversed View Mechanics
`reversed()` provides a view whose iteration order is the opposite of the underlying structure; structural modifications reconcile in constant time via internal links (LinkedHash*). For `ArrayList`, the reversed view calculates indices offset; operations convert logically.

### 2.2 Complexity & Performance
| Operation | ArrayList | LinkedHashSet | LinkedHashMap |
|-----------|-----------|---------------|---------------|
| `getFirst()` | O(1) index 0 | O(1) head pointer | O(1) head entry |
| `getLast()` | O(1) index size-1 | O(1) tail pointer | O(1) tail entry |
| `addFirst()` | O(n) shift for ArrayList | O(1) (re-link head) | N/A (Map uses put preserving order) |
| `addLast()` | Amortized O(1) | O(1) | O(1) |
| `removeFirst()/removeLast()` | O(n) shift for ArrayList | O(1) | O(1) |

Note: Endpoint removal in `ArrayList` may require shifting unless removing last.

### 2.3 SequencedSet Re-add Behavior
Re-adding existing element returns false; order remains unchanged preventing implicit repositioning. To move element you must remove then re-add.

### 2.4 Polling vs Removal
- `removeFirst()` / `removeLast()` throw `NoSuchElementException` on empty.
- `pollFirstEntry()` / `pollLastEntry()` return null if empty (non-exceptional). Fits map patterns from existing `NavigableMap` APIs.

### 2.5 Integration with Streams & Pipelines
Endpoint access avoids costly operations:
```java
var list = new ArrayList<>(List.of(1,2,3,4,5));
int first = list.getFirst(); // direct
int last  = list.getLast();
var middle = list.reversed().stream().skip(1).limit(list.size()-2).toList();
```

### 2.6 Comparison with Deque
Deque lacks reversed live view and unified map/set semantics. Sequenced interfaces reduce need for multiple method families (`offerFirst`, etc.) when only ordered endpoints needed.

### 2.7 Concurrency Considerations
Live reversed views share underlying mod counts—unsynchronized concurrent modifications can trigger `ConcurrentModificationException`. Use wrappers or copies for multi-threaded scenarios.

### 2.8 API Migration Strategy
Refactor method signatures from `List<E>` to `SequencedCollection<E>` when only ordering + endpoints required. Improves polymorphism (accepts `ArrayList`, future types).

### 2.9 Fallback Patterns (Pre-Java 21)
If backport required, supply helper interface + adapter. (Legacy note; not needed on 21+ but relevant for libraries supporting older JVMs.)

### 2.10 Memory Considerations
Reversed view avoids allocation of an entire reversed copy; only lightweight view object.

### 2.11 Example: Reordering via Remove/Re-add
```java
SequencedSet<String> tags = new LinkedHashSet<>(List.of("alpha","beta","gamma"));
tags.remove("beta");
tags.addFirst("beta"); // now order: beta, alpha, gamma
```

### 2.12 Exercise Set (In-Depth)
1. Measure difference between `removeFirst()` on `ArrayList` vs `LinkedHashSet`.
2. Demonstrate reordering element in `SequencedSet`.
3. Use reversed view to implement back-to-front search.
4. Implement cache eviction using `pollFirstEntry()`.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Primary purpose of SequencedCollection? | Sorting elements | Unified endpoint ops & reversed view | Thread safety | Randomization | B | Adds first/last + reversed API universally. |
| 2 | `reversed()` returns? | Immutable snapshot | Live reversed view | Synchronized wrapper | Copy with reverse order | B | Mutations reflect both views. |
| 3 | Empty `removeFirst()` on SequencedCollection throws? | NullPointerException | NoSuchElementException | IllegalStateException | Returns null | B | Contracts specify exception on empty. |
| 4 | `pollLastEntry()` on empty SequencedMap returns? | null | throws | Optional.empty() | false | A | Poll methods return null if absent. |
| 5 | Re-adding existing element to SequencedSet using `addLast()`? | Moves to end | No effect (returns false) | Throws | Replaces value | B | Set uniqueness prevents reposition. |
| 6 | Complexity of `getLast()` in ArrayList? | O(n) | O(1) | O(log n) | Depends on size | B | Direct index access. |
| 7 | To move an existing element to front in SequencedSet you must? | Use addFirst directly | Remove then addFirst | Use reversed().addLast | Not possible | B | Need remove + re-add. |
| 8 | Which supports SequencedMap in Java 21? | HashMap | LinkedHashMap | TreeMap | ConcurrentHashMap | B | LinkedHashMap implements ordering. |
| 9 | Benefit over Deque? | Deque faster | Sequenced has reversed live view | Deque uses less memory | Deque supports maps | B | Reversed view unique to sequenced. |
| 10 | `firstEntry()` complexity in LinkedHashMap? | O(n) | O(1) | O(log n) | Amortized O(n) | B | Maintained pointer. |
| 11 | Null key allowance depends on? | SequencedMap forbids null | Underlying implementation | Always allowed | JIT optimization | B | Follows underlying map behavior. |
| 12 | Reversed view memory overhead? | Large copy | None (lazy) | Doubles memory | Forces GC | B | Lightweight live view. |
| 13 | Endpoint removal on `ArrayList.removeFirst()` cost? | O(1) | O(n) shift | O(log n) | random | B | Shifts remaining elements. |
| 14 | Safe concurrent iteration strategy? | Use reversed() always | Synchronize or snapshot copy | Disable GC | Convert to array each access | B | Need external synchronization or snapshot. |
| 15 | `pollFirstEntry()` usefulness in cache? | Evicts oldest entry | Sorts entries | Random removal | Locks map | A | Facilitates FIFO/LRU eviction.

### MCQ Extension Exercise
Draft 2 MCQs: one about reordering in SequencedSet, one about reversed view concurrency risk.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Endpoint Operations
Create `SequencedCollection<Integer>`; perform add/remove first & last; print states.

### Exercise 4.2: Live Reversed Mutation
Given list `[A,B,C]`, use reversed view to append at front and show original change.

### Exercise 4.3: Cache Eviction
Use `LinkedHashMap` as `SequencedMap`; remove oldest when size exceeds threshold.

### Exercise 4.4: Reordering Set Element
Move an existing element "beta" to front in `SequencedSet`.

### Exercise 4.5: Back-to-Front Search
Search for target scanning reversed view early exit when found.

### Exercise 4.6: Compare Deque vs SequencedCollection
Show equivalent code for endpoint operations and reversed iteration.

### Exercise 4.7: Middle Slice via Reversed
Extract all but first & last using reversed view and stream slicing.

### Exercise 4.8: Micro Benchmark Skeleton
Outline JMH benchmark for `getLast()` vs manual iteration.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Using Sequenced APIs on < Java 21
Compilation error: symbol not found. Ensure proper JDK level.

### 5.2 Misinterpreting Reversed View
Assuming independence leads to unexpected original mutation side-effects.

### 5.3 Removing From Empty Collection
Calling `removeFirst()` on empty results in `NoSuchElementException`; must guard.

### 5.4 Reordering via addLast
Expecting reposition of existing element in `SequencedSet` fails silently (false return).

### 5.5 Concurrency Without Snapshot
Concurrent modifications while iterating may throw `ConcurrentModificationException`.

### 5.6 Exercise: Identify Errors
```java
SequencedSet<String> s = new LinkedHashSet<>();
s.removeFirst(); // Error: NoSuchElementException (empty)
s.addLast("x");
s.addLast("x"); // returns false, no reposition
SequencedCollection<String> sc = new ArrayList<>();
sc.reversed().addFirst("z"); // valid but may confuse direction semantics
```
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 LRU Cache with SequencedMap
Use insertion-order `LinkedHashMap` with size checks and `pollFirstEntry()` for eviction.

### 6.2 Backward Traversal Without Copy
Use `reversed()` for tail-first processing (e.g., recent events first).

### 6.3 Unified API Layer
Accept `SequencedCollection<E>` in service methods abstracting from concrete list/set types while retaining order semantics.

### 6.4 Reordering Strategy
Remove + addFirst to reposition existing element in a set.

### 6.5 Efficient Endpoint Checks
Constant-time first/last access avoids overhead of iteration or index math wrappers.

### 6.6 Snapshotting for Concurrency
Create `new ArrayList<>(sc)` before parallel processing tasks.

### 6.7 Filter Tail Elements
```java
List<String> tailFiltered = sc.reversed().stream()
    .takeWhile(s -> !s.equals("STOP"))
    .toList();
```

### 6.8 Range Extraction
Use combination of `skip`/`limit` on reversed/forward streams to isolate middle ranges.

### 6.9 Multi-View Coordination
Maintain forward and reversed views for algorithm comparing head vs tail simultaneously without manual index arithmetic.

### 6.10 Exercise: Implement size-limited event buffer using `SequencedCollection` removing oldest when capacity exceeded.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. What distinguishes SequencedCollection from List?  
List offers random access; SequencedCollection adds standardized endpoint & reversed operations across multiple ordered collection families.
2. How does reversed view behave under mutation?  
Live; modification in either view reflects in the other.
3. Why not rely solely on Deque?  
Deque lacks reversed view and does not apply to sets/maps uniformly.
4. How do SequencedSet operations differ from List regarding duplicates?  
Set uniqueness prevents duplicates; re-adding existing element does not reposition.
5. When use `pollFirstEntry()` vs `removeFirst()`?  
Use poll variants on maps to avoid exceptions when empty.
6. Performance benefits?  
Direct O(1) endpoint access enabling simpler algorithms vs manual iteration or index computations.
7. Migration approach to new interfaces?  
Refactor method signatures and internal abstractions where only ordering + endpoints needed.
8. Concurrency caution?  
Live views share structure; snapshot or synchronize to avoid CME.
9. Reordering elements in SequencedSet?  
Remove then addFirst/addLast; cannot reposition by re-adding.
10. Memory advantage of reversed view?  
No full copy allocation; lightweight view object.

### Exercise: Explain algorithmic simplification using `reversed()` for tail-first processing.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
(1–4) Code snippets demonstrating endpoint operations, reversed view, uniqueness check, map endpoints.

## Solutions: Module 2 Exercises
Show timing difference (conceptual) & reordering via remove/addFirst, back-to-front search, cache eviction using poll.

## Solutions: Module 3 MCQ Extension
1. Reordering requires explicit remove + addFirst.
2. Concurrency risk: iteration may throw CME if underlying mutated concurrently.

## Solutions: Module 5 Error Identification (5.6)
Calling removeFirst on empty triggers exception; duplicate add returns false (no reposition); reversed direction semantics clarified.

## Solutions: Module 6 Exercise
Event buffer: on add if size > capacity then removeFirst.

## Solutions: Module 7 Exercise
Tail-first processing avoids index arithmetic, uses reversed() for natural ordering of recent items.

---

### End Notes
Sequenced Collections unify and simplify ordered data manipulation in Java 21. Adopt them to reduce boilerplate, clarify intent, and enable efficient endpoint-oriented algorithms without sacrificing compatibility with existing collection types.
