# Java Method References – In-Depth Training

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

---
## Table of Contents
1. [Module 1 – Basics & Theory](#module-1--basics--theory)
2. [Module 2 – In-Depth Concepts](#module-2--in-depth-concepts)
3. [Module 3 – Multiple Choice Questions (MCQs)](#module-3--multiple-choice-questions-mcqs)
4. [Module 4 – Practical Hands-On Exercises](#module-4--practical-hands-on-exercises)
5. [Module 5 – Common Compilation & Runtime Errors](#module-5--common-compilation--runtime-errors)
6. [Module 6 – Techniques, Patterns & Hacks](#module-6--techniques-patterns--hacks)
7. [Module 7 – Interview Questions & Answers](#module-7--interview-questions--answers)
8. [Module 8 – Solutions Appendix](#module-8--solutions-appendix)

---
<div style="page-break-before: always;"></div>

# Module 1 – Basics & Theory

### 1.1 What Are Method References?
Method references provide a shorthand syntax to refer to existing methods or constructors, enabling concise lambda expressions when just forwarding arguments.
They are syntactic sugar over lambdas that simply call a method.

### 1.2 Four Primary Forms
| Form | Syntax | Example | Equivalent Lambda |
|------|--------|---------|-------------------|
| Static method | `ClassName::staticMethod` | `Integer::parseInt` | `s -> Integer.parseInt(s)` |
| Instance method of particular object | `instanceRef::instanceMethod` | `printer::println` | `x -> printer.println(x)` |
| Instance method of arbitrary object of a particular type | `Type::instanceMethod` | `String::toLowerCase` | `s -> s.toLowerCase()` |
| Constructor reference | `ClassName::new` | `ArrayList::new` | `() -> new ArrayList<>()` |

### 1.3 ASCII Concept Diagram
```
Lambda: (args) -> target.method(args)
          |                ^
          +----------------+
Method Reference: target::method
```

### 1.4 Functional Interface Compatibility
A method reference matches a functional interface if its signature is compatible with the interface's single abstract method via parameter adaptation and return type covariance.

Example:
```java
Function<String, Integer> f = Integer::parseInt; // parseInt(String) -> int auto-boxed
Supplier<List<String>> sup = ArrayList::new;     // matches Supplier<List<String>>
```

### 1.5 Replacement Mechanics
The compiler infers types from context (target typing). If ambiguous, explicit types or lambdas may be necessary.

### 1.6 Simple Examples
```java
List<String> nums = List.of("10","20","30");
List<Integer> ints = nums.stream().map(Integer::parseInt).toList();

List<String> words = List.of("A","B","C");
words.forEach(System.out::println);

Stream.of("alpha","beta")
    .map(String::length)
    .forEach(System.out::println);
```

### 1.7 Constructor References with Arguments
```java
interface Maker<T,U> { T make(U u); }
class Box { final String value; Box(String v){ this.value=v; } }
Maker<Box,String> maker = Box::new; // matches (String)->Box
Box b = maker.make("Hi");
```

### 1.8 Array Constructor References
```java
IntFunction<String[]> arrayMaker = String[]::new; // length -> new String[length]
String[] arr = arrayMaker.apply(5);
```

### 1.9 Bound vs Unbound
- Bound reference: refers to specific instance `obj::method`.
- Unbound reference: `Type::method` expects receiver as first parameter implicitly.

Example:
```java
Consumer<String> printer = System.out::println; // bound to System.out
Function<String, String> lower = String::toLowerCase; // unbound; receiver becomes parameter
```

### 1.10 Exercise Set (Basics)
1. Convert lambda `s -> s.trim()` to method reference.
2. Use `PrintStream` instance to create a `Consumer<String>` via method reference.
3. Replace `s -> new BigDecimal(s)` with a constructor reference.
4. Map a list of files to paths using method reference.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Concepts

### 2.1 Type Inference & Target Typing
The compiler determines the functional interface from assignment, method arguments, or return context. Method reference must match erased and boxed types appropriately.

### 2.2 Overload Resolution
When multiple overloaded methods could match, the compiler uses context to choose; ambiguous references cause errors requiring explicit casts.
```java
class Over {
    static void m(int x){}
    static void m(Integer x){}
}
Consumer<Integer> c = Over::m; // picks m(Integer)
IntConsumer ic = Over::m;      // picks m(int)
```

### 2.3 Generic Methods
```java
class Util {
    static <T> T id(T t){ return t; }
}
Function<String,String> f = Util::id; // type inference selects <String>
```

### 2.4 Capturing vs Non-Capturing
Method references do not capture additional variables beyond the receiver; for capturing, use lambda.
```java
String prefix = "P:";
Function<String,String> f1 = prefix::concat; // OK bound to prefix instance
Function<String,String> f2 = s -> prefix + s; // lambda for concatenation pattern
```

### 2.5 Reference to Private Methods
Possible within same class context using instance or class reference; accessibility rules still apply.

### 2.6 Performance Considerations
Method references compile to invokedynamic instructions similarly to lambdas; no guaranteed performance difference. Choose for readability.

### 2.7 Chaining with Streams
```java
List<Person> people = ...;
List<String> names = people.stream().map(Person::getName).sorted().toList();
```

### 2.8 Constructor Reference with Multiple Params
```java
record Pair<A,B>(A a, B b) {}
BiFunction<String,Integer, Pair<String,Integer>> pf = Pair::new;
Pair<String,Integer> p = pf.apply("Age", 30);
```

### 2.9 Array References in Mapping
```java
String[][] grid = Stream.generate(() -> new String[]{"x","y"}).limit(3).toArray(String[][]::new);
```

### 2.10 Edge Ambiguity Case
```java
class Amb {
    static void f(Number n){}
    static void f(Integer n){}
}
Consumer<Integer> ci = Amb::f; // picks f(Integer)
Consumer<Number> cn = Amb::f;  // picks f(Number)
// BiConsumer<Integer,Integer> bc = Amb::f; // Compile error: no match
```

### 2.11 Method Reference vs Lambda Choice
Choose method reference when it communicates intent clearly and reduces noise; fallback to lambda for clarity when transformation different from just calling a method.

### 2.12 Exercise Set (In-Depth)
1. Resolve an ambiguous overload by casting method reference.
2. Create a generic method reference to identity function used in a stream.
3. Use constructor reference for record with three fields.
4. Replace lambda performing `File::getName` mapping.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Method reference replacing lambda `s -> s.toUpperCase()`? | `String::toUpperCase` | `s::toUpperCase` | `Object::toUpper` | `String::upper` | A | `String::toUpperCase` maps each element's receiver. |
| 2 | Constructor reference syntax? | `new::Class` | `Class::new` | `Class.new()` | `Class:new` | B | `Class::new` is correct. |
| 3 | Static method reference of `Math.max`? | `Math::max` | `max::Math` | `Math.max::()` | `Math::max()` | A | Remove parentheses. |
| 4 | Equivalent of `System.out::println`? | `s -> System.out.println(s)` | `() -> System.out.println()` | `System::println` | `Printer::println` | A | Accepts argument forwarded. |
| 5 | Unbound instance reference example? | `this::length` | `str::trim` | `String::length` | `out::println` | C | `String::length` expects receiver parameter. |
| 6 | Array constructor reference returning `String[]`? | `String::new[]` | `String[]::new` | `new::String[]` | `String.array::new` | B | Syntax is `Type[]::new`. |
| 7 | Method reference vs lambda performance? | Always faster | Always slower | Usually same | Creates thread | C | Both compile to similar invokedynamic constructs. |
| 8 | Overload resolution uses? | Run-time dispatch | Compile-time typing | Reflection | Random selection | B | Compiler chooses based on target type. |
| 9 | Method reference capturing additional local vars? | Yes automatically | Only static | No (unless bound) | Via reflection | C | Only bound receiver captured. |
| 10 | Record constructor reference of `Point(int x,int y)`? | `Point::new` | `new::Point` | `Point.new::()` | `Point:new` | A | `Point::new` matches BiFunction. |
| 11 | `String::compareToIgnoreCase` target functional interface? | `Comparator<String>` | `BiPredicate<String,String>` | `BiFunction<String,String,Integer>` | `Predicate<String>` | C | compareToIgnoreCase returns int; two params. |
| 12 | Static method reference chooses overload by? | Return type only | Parameter types in target functional interface | Variable names | Order defined | B | Parameter types drive selection. |
| 13 | Context where ambiguity occurs? | Unique signature | Multiple overloads matching | No methods | Private method only | B | Multiple overloads can match. |
| 14 | Bound reference example? | `someList::size` | `List::size` | `String::length` | `Integer::parseInt` | A | Bound to instance variable someList. |
| 15 | Constructor reference for array of Person of length n? | `Person[]::new` | `Person::[]new` | `new::Person[]` | `Person.array::new` | A | `Person[]::new` correct.

### MCQ Extension Exercise
Write 2 more MCQs: one about generic method references, one about record constructor references.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: Parsing Integers
Map list of numeric strings to integers using method reference.

### Exercise 4.2: System Output Binding
Bind `System.out::println` to a `Consumer<String>` and use in stream pipeline.

### Exercise 4.3: Record Construction
Create a record `User(String name, int age)` and construct instances from a list of pairs using `User::new`.

### Exercise 4.4: Array Generation
Use `String[]::new` in `toArray` to create typed array from stream.

### Exercise 4.5: Reduce with Method Reference
Sum integers using `Integer::sum` or `Math::max` to find maximum.

### Exercise 4.6: Custom Factory Interface
Define functional interface taking two ints returning `Point` using `Point::new`.

### Exercise 4.7: Overload Cast Resolution
Resolve ambiguous `valueOf` method reference via explicit cast to `Function<String,Integer>`.

### Exercise 4.8: Identity Generic Method
Use `Util::id` in mapping a list to itself unchanged.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Ambiguous Overload
```java
class V {
    static int parse(String s){ return Integer.parseInt(s); }
    static Integer parse(Object o){ return Integer.parseInt(o.toString()); }
}
// Function<String,Integer> f = V::parse; // Ambiguous? picks most specific; OK
// BiFunction<String,String,Integer> bad = V::parse; // Error: no matching signature
```

### 5.2 Wrong Constructor Arity
```java
record Pair(String a, int b){}
Supplier<Pair> s = Pair::new; // Error: constructor needs (String,int)
```
Correct:
```java
BiFunction<String,Integer, Pair> bf = Pair::new;
```

### 5.3 Instance vs Static Confusion
```java
Function<String,Integer> f = Integer::intValue; // Error: needs instance method on Integer
```
Correct usage:
```java
ToIntFunction<Integer> g = Integer::intValue;
```

### 5.4 Incompatible Return Type
```java
Function<String, Long> f = Integer::parseInt; // Error: expects Long not int
```

### 5.5 Private Method Access
```java
class Hidden { private void go(){} }
Runnable r = new Hidden()::go; // Error: go() not accessible
```

### 5.6 Array Constructor Wrong Type
```java
IntFunction<List<String>> maker = String[]::new; // Error: returns String[] not List<String>
```

### 5.7 Capturing Non-Final Variable (Lambda Only Case)
```java
int x = 10; // mutated later
Supplier<Integer> s = () -> x; // allowed if effectively final; method reference can't adapt different variable usage scenario requiring modification
```

### 5.8 Exercise: Identify Errors
```java
record Person(String name,int age){}
Supplier<Person> sp = Person::new;
Function<Integer, Person> fp = Person::new;
BiFunction<String,Integer, Person> bp = Person::new;
```
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Cleaner Pipelines
Replace verbose lambdas with method references for readability.

### 6.2 Combining With Optional
```java
Optional<String> opt = Optional.of("value");
opt.map(String::length).ifPresent(System.out::println);
```

### 6.3 Constructor References for Factory Registries
Store constructor references in maps for dynamic instantiation.
```java
Map<String, Supplier<List<String>>> registry = Map.of(
    "arrayList", ArrayList::new,
    "linkedList", LinkedList::new
);
List<String> list = registry.get("arrayList").get();
```

### 6.4 Method Reference for Currying-Like Patterns
Bind instance to get specialized function:
```java
String prefix = "LOG:";
Function<String,String> addPrefix = prefix::concat;
```

### 6.5 Array Projection Helper
Use array constructor in reflective operations.

### 6.6 Parallel Stream Utility
```java
long sum = IntStream.rangeClosed(1,1000).parallel().reduce(0, Integer::sum);
```

### 6.7 Validation via Reference
Reference existing static validators directly instead of wrapping lambdas.

### 6.8 Collector Combination
```java
List<String> upper = words.stream().map(String::toUpperCase).toList();
```

### 6.9 Generic Identity Usage
```java
Function<String,String> id = Util::id; // reuse across pipeline
```

### 6.10 Exercise: Build registry using record constructor references.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Difference between bound and unbound method reference?  
Bound ties to a specific instance; unbound expects receiver as first parameter.
2. When would you prefer lambda over method reference?  
Custom logic, parameter reordering, additional statements.
3. Do method references improve performance?  
Usually no; they improve readability only.
4. How does constructor reference perform type inference?  
Compiler matches target functional interface parameter types to constructor parameters.
5. How to handle overload ambiguity?  
Provide explicit target type or cast to functional interface to guide resolution.
6. Can method references refer to generic methods?  
Yes; type parameters inferred from context (e.g., `Util::id`).
7. Are method references compatible with primitive functional interfaces?  
Yes (`IntConsumer`, `ToIntFunction` etc.) if signatures match.
8. How do array constructor references work?  
`Type[]::new` takes array length returning new array; fits `IntFunction<Type[]>`.
9. How to reference instance method requiring no params?  
Use `instanceRef::methodName` producing Supplier-like interface if return matches.
10. Can you reference a private method outside its class?  
No; access control applies.

### Exercise: Draft explanation of overload resolution with example.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. `String::trim`
2. `PrintStream ps = System.out; Consumer<String> c = ps::println;`
3. `BigDecimal::new`
4. `files.stream().map(File::toPath)...`

## Solutions: Module 2 Exercises
1. Cast: `(Function<String,Integer>) Integer::valueOf`
2. Identity: `Function<String,String> id = Util::id; list.stream().map(Util::id)...`
3. Record: `record Triple(String a,int b,double c){} TriFunction<String,Integer,Double, Triple> t = Triple::new;`
4. `map(File::getName)`

## Solutions: Module 3 MCQ Extension
1. Generic method reference: `Function<Integer,Integer> f = Util::id;` Explanation: `<Integer>` inferred.
2. Record constructor reference: `BiFunction<String,Integer,Person> bf = Person::new;` Explanation: matches params.

## Solutions: Module 4 Selected Exercises
Exercise 4.1:
```java
List<Integer> ints = nums.stream().map(Integer::parseInt).toList();
```
Exercise 4.2:
```java
Consumer<String> out = System.out::println;
words.forEach(out);
```
Exercise 4.3:
```java
record User(String name,int age){}
List<User> users = pairs.stream().map(p -> new User(p.name(), p.age())).toList(); // or p -> new User(...)
```
Exercise 4.5 (Max):
```java
int max = ints.stream().reduce(Integer.MIN_VALUE, Integer::max);
```
Exercise 4.6:
```java
record Point(int x,int y){}
BiFunction<Integer,Integer,Point> pf = Point::new;
```
Exercise 4.7:
```java
Function<String,Integer> f = (Function<String,Integer>) Integer::valueOf;
```
Exercise 4.8:
```java
list.stream().map(Util::id).toList();
```

## Solutions: Module 5 Error Identification (5.8)
Errors:
1. `Supplier<Person>` invalid: constructor needs params.
2. `Function<Integer, Person>` invalid: constructor needs (String,int) not (Integer).
3. `BiFunction<String,Integer, Person>` correct.

## Solutions: Module 7 Exercise
Overload resolution selects the most specific signature compatible with target type. Example: `Consumer<Integer> c = Amb::f;` chooses `f(Integer)` over `f(Number)` due to specificity.

---

### End Notes
Method references are a readability enhancement—favor them when they clarify intent without hiding logic. Fall back to lambdas for complex transformations.
