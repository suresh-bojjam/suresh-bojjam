# Java Native Interface (JNI) – Comprehensive Training (Java 8 Perspective, Modern Practices up to Java 21)

> Date: 2025-10-28  
> Author: Senior Technical Java Trainer & Staff Software Engineer

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

### 1.1 What is JNI?
JNI is a programming framework enabling Java code running in the JVM to call and be called by native applications and libraries written in C, C++, and assembly. It acts as a bridge for legacy integration, performance hotspots, and OS/platform-specific features.

### 1.2 When to Use JNI
- Access native system APIs not exposed in pure Java.
- Integrate existing optimized C/C++ libraries.
- Leverage SIMD or GPU interfaces through wrapper libraries.
- Performance-critical sections where Java overhead is unsuitable.

Avoid JNI for general application logic; prefer pure Java for portability, safety, and maintainability.

### 1.3 High-Level Architecture
```
+------------------+        +----------------------+
|   Java Code      | <----> |  Native Library (.so/.dll/.dylib) |
+---------+--------+        +-------------+--------+
          | JNI Bridge                      |
          v                                 v
    JVM Runtime                     OS / Hardware APIs
```

### 1.4 Basic Workflow
1. Declare native method in Java class.
2. Load native library (`System.loadLibrary("mylib")`).
3. Generate C/C++ header (historically `javah`, now `javac -h`).
4. Implement C/C++ function with correct signature.
5. Compile to shared library.
6. Run Java application; JVM resolves native symbol.

### 1.5 Declaring a Native Method
```java
public class NativeMath {
    static { System.loadLibrary("nativemath"); }
    public native int add(int a, int b);
}
```

### 1.6 Generating Header (Java 8+)
```bash
javac -h native NativeMath.java
```
Produces `native/NativeMath.h` containing JNI-compliant function signature.

### 1.7 Anatomy of a JNI Function
Generated signature (simplified example):
```c
JNIEXPORT jint JNICALL Java_NativeMath_add(JNIEnv* env, jobject obj, jint a, jint b) {
    return a + b;
}
```
- `JNIEnv* env`: Interface pointer providing JNI functions (like FindClass, NewObject, etc.).
- `jobject obj`: Reference to calling Java object (for instance methods). For static methods use `jclass`.
- Types: `jint`, `jlong`, `jstring`, `jobjectArray`, etc. Map to Java primitives and reference types.

### 1.8 Mapping Types
| Java Type | JNI Type | Notes |
|----------|----------|-------|
| int | jint | 32-bit signed |
| long | jlong | 64-bit signed |
| boolean | jboolean | Unsigned 8-bit (JNI_TRUE/JNI_FALSE) |
| double | jdouble | 64-bit IEEE |
| byte[] | jbyteArray | Use `GetByteArrayElements` |
| String | jstring | Use `GetStringUTFChars` (careful encoding) |

### 1.9 Reference vs Value Semantics
JNI uses opaque handles for objects; you manipulate through the `JNIEnv` methods. Primitive values pass directly; objects require method calls to access fields/elements.

### 1.10 Local vs Global References
- Local: Auto-freed when native frame returns; bounded by local ref table capacity.
- Global: Created via `NewGlobalRef`; must be manually deleted with `DeleteGlobalRef`.

### 1.11 Garbage Collection Interaction
The JVM tracks reachability through references held in JNI tables. Failing to delete global references -> memory leaks in long-running processes.

### 1.12 Exercise Set (Basics)
1. Write a class declaring a native `multiply(int,int)` method.
2. Generate header using `javac -h`.
3. Implement addition & multiplication in C.
4. Load library and invoke; print result.

(See Module 8 for solutions.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Accessing Fields
```c
jclass cls = (*env)->GetObjectClass(env, obj);
jfieldID fid = (*env)->GetFieldID(env, cls, "counter", "I");
jint counter = (*env)->GetIntField(env, obj, fid);
(*env)->SetIntField(env, obj, fid, counter + 1);
```

### 2.2 Calling Java Methods from Native
```c
jmethodID mid = (*env)->GetMethodID(env, cls, "notifyUpdate", "(I)V");
(*env)->CallVoidMethod(env, obj, mid, counter);
```

### 2.3 Working with Arrays
```c
jintArray arr = /* passed from Java */;
jsize len = (*env)->GetArrayLength(env, arr);
jint* elems = (*env)->GetIntArrayElements(env, arr, NULL);
for (jsize i = 0; i < len; i++) { elems[i] *= 2; }
(*env)->ReleaseIntArrayElements(env, arr, elems, 0); // commit changes
```

### 2.4 Strings and Encoding
Use `GetStringUTFChars` or `GetStringChars` (for UTF-16). Always release.
```c
const char* utf = (*env)->GetStringUTFChars(env, jstr, NULL);
// ... use utf ...
(*env)->ReleaseStringUTFChars(env, jstr, utf);
```

### 2.5 Exception Handling
Check for exceptions after calls that can throw:
```c
if ((*env)->ExceptionCheck(env)) {
    (*env)->ExceptionDescribe(env);
    (*env)->ExceptionClear(env);
    // Optionally throw a new exception back to Java
}
```
Throwing:
```c
jclass exCls = (*env)->FindClass(env, "java/lang/IllegalArgumentException");
(*env)->ThrowNew(env, exCls, "Invalid argument");
```

### 2.6 Creating Objects
```c
jclass pointCls = (*env)->FindClass(env, "com/example/Point");
jmethodID ctor = (*env)->GetMethodID(env, pointCls, "<init>", "(II)V");
jobject point = (*env)->NewObject(env, pointCls, ctor, 10, 20);
```

### 2.7 Global & Weak Global References
```c
jobject gref = (*env)->NewGlobalRef(env, point);
// Later
(*env)->DeleteGlobalRef(env, gref);
```
Weak global references allow GC collection if only weak refs remain: `NewWeakGlobalRef`.

### 2.8 Performance Considerations
- Minimize transitions across JNI boundary.
- Batch operations (e.g., process entire array in C rather than per-element callbacks).
- Avoid frequent creation of local refs in loops; use `PushLocalFrame`/`PopLocalFrame` for bulk allocation.

### 2.9 Multi-Threading & Attach/Detach
Native threads must attach to JVM before using JNI:
```c
JavaVM* jvm; // acquired via JNI_OnLoad
JNIEnv* env;
(*jvm)->AttachCurrentThread(jvm, (void**)&env, NULL);
// ... work ...
(*jvm)->DetachCurrentThread(jvm);
```

### 2.10 JNI_OnLoad
Called when library loaded. Cache class/method IDs.
```c
JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved) {
    JNIEnv* env = NULL;
    if ((*vm)->GetEnv(vm, (void**)&env, JNI_VERSION_1_8) != JNI_OK) return JNI_ERR;
    // Cache references if needed
    return JNI_VERSION_1_8;
}
```

### 2.11 Security Considerations
- Validate input sizes before allocating native buffers.
- Avoid buffer overflows (use `jsize len` values carefully).
- Prefer safer functions (`memcpy` with bounds) and static analysis.

### 2.12 Memory Management Pitfalls
Not releasing array or string elements yields pinned memory; can degrade GC performance.

### 2.13 Exercise Set (In-Depth)
1. Access and increment a Java field from native.
2. Call back into a Java method from C after processing array.
3. Create a Java object in native and return it.
4. Implement `JNI_OnLoad` caching a class reference.
5. Attach a new native thread and call a method.

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Tool to generate headers (Java 8+) | javap | javah | javac -h | jlink | C | `javac -h` replaced `javah`. |
| 2 | JNI pointer providing functions? | JCL | JNIEnv* | JavaVMOption | JMethod | B | `JNIEnv*` gives access to JNI funcs. |
| 3 | Type mapping for Java long? | jlong | jint | jsize | jlonglong | A | `jlong` is 64-bit. |
| 4 | Must manually release after `GetStringUTFChars`? | No | Only on error | Yes | Only for long strings | C | Release to avoid leaks. |
| 5 | Creating global ref function? | NewGlobalRef | MakeGlobal | GlobalRefCreate | NewRefGlobal | A | Correct JNI call. |
| 6 | Safe to ignore `ExceptionCheck`? | Yes | Only if no calls made | No | Only in Java 21 | C | Always check to avoid silent failure. |
| 7 | Local refs lifetime? | Until GC | Until native frame returns | Forever | After PopLocalFrame only | B | Auto-freed when method returns. |
| 8 | Which reduces overhead? | Many tiny JNI calls | Batch operations | Frequent attach/detach | Using reflection every call | B | Batch reduces boundary crossings. |
| 9 | Attach native thread method? | AttachCurrentThread | LinkThread | RegisterThread | BindThread | A | Proper JNI call. |
| 10 | Return version in `JNI_OnLoad`? | JNI_VERSION_1_8 | 8 | 1.8 | JNI_VERSION_LATEST | A | Use constant. |
| 11 | If failing to delete global refs? | Java crash | GC ignores them | Memory leak | Optimized performance | C | They leak memory. |
| 12 | Acquire array length? | GetArraySize | GetArrayLength | ArrayLength | LengthOfArray | B | Correct API. |
| 13 | To call static Java method use? | CallVoidMethod | CallStaticVoidMethod | CallMethodStatic | InvokeStatic | B | Static variant. |
| 14 | Multi-threading requirement? | Each thread auto attached | Manual attach required | Only main thread works | Use Thread.start | B | Must attach manually. |
| 15 | Passing large arrays efficiently? | Element by element calls | Pin & process in bulk | Convert to string | Impossible | B | Bulk processing better. |

### MCQ Extension
Add two more:
16. Returning object from native requires? (A: NewObject)  
17. Cache class/method IDs where? (A: JNI_OnLoad)

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Learning Examples

### 4.1 Full Add & Multiply Example
`NativeMath.java`:
```java
public class NativeMath {
    static { System.loadLibrary("nativemath"); }
    public native int add(int a, int b);
    public native int multiply(int a, int b);
    public static void main(String[] args) {
        NativeMath nm = new NativeMath();
        System.out.println("Add: " + nm.add(3,4));
        System.out.println("Multiply: " + nm.multiply(3,4));
    }
}
```
Header (simplified):
```c
JNIEXPORT jint JNICALL Java_NativeMath_add(JNIEnv*, jobject, jint, jint);
JNIEXPORT jint JNICALL Java_NativeMath_multiply(JNIEnv*, jobject, jint, jint);
```
Implementation:
```c
JNIEXPORT jint JNICALL Java_NativeMath_add(JNIEnv* env, jobject obj, jint a, jint b) { return a + b; }
JNIEXPORT jint JNICALL Java_NativeMath_multiply(JNIEnv* env, jobject obj, jint a, jint b) { return a * b; }
```

### 4.2 Field Access & Callback
```java
public class Counter {
    static { System.loadLibrary("counter"); }
    private int counter = 0;
    public native void incrementAndNotify();
    private void notifyUpdate(int value) { System.out.println("Counter=" + value); }
}
```
Native C:
```c
JNIEXPORT void JNICALL Java_Counter_incrementAndNotify(JNIEnv* env, jobject obj) {
    jclass cls = (*env)->GetObjectClass(env, obj);
    jfieldID fid = (*env)->GetFieldID(env, cls, "counter", "I");
    jint val = (*env)->GetIntField(env, obj, fid);
    (*env)->SetIntField(env, obj, fid, val + 1);
    jmethodID mid = (*env)->GetMethodID(env, cls, "notifyUpdate", "(I)V");
    (*env)->CallVoidMethod(env, obj, mid, val + 1);
}
```

### 4.3 Array Processing
```java
public class ArrayOps {
    static { System.loadLibrary("arrops"); }
    public native int[] doubleValues(int[] in);
}
```
Native:
```c
JNIEXPORT jintArray JNICALL Java_ArrayOps_doubleValues(JNIEnv* env, jobject obj, jintArray in) {
    jsize len = (*env)->GetArrayLength(env, in);
    jint* elems = (*env)->GetIntArrayElements(env, in, NULL);
    for (jsize i=0;i<len;i++) elems[i]*=2;
    (*env)->ReleaseIntArrayElements(env, in, elems, 0);
    return in; // modified in place
}
```

### 4.4 Creating Java Objects in Native
```java
public class PointFactory {
    static { System.loadLibrary("pointfactory"); }
    public static class Point { public final int x,y; public Point(int x,int y){this.x=x;this.y=y;} }
    public native Point makePoint(int x,int y);
}
```
Native:
```c
JNIEXPORT jobject JNICALL Java_PointFactory_makePoint(JNIEnv* env, jobject obj, jint x, jint y) {
    jclass outerCls = (*env)->GetObjectClass(env, obj);
    jclass pointCls = (*env)->FindClass(env, "PointFactory$Point");
    jmethodID ctor = (*env)->GetMethodID(env, pointCls, "<init>", "(II)V");
    return (*env)->NewObject(env, pointCls, ctor, x, y);
}
```

### 4.5 Exception Propagation
Throw native validation failure.
```c
if (x < 0 || y < 0) {
    jclass iae = (*env)->FindClass(env, "java/lang/IllegalArgumentException");
    (*env)->ThrowNew(env, iae, "Coordinates must be non-negative");
    return NULL;
}
```

### 4.6 Attaching Threads
Demonstrate background worker calling back.

### 4.7 Using `PushLocalFrame`
```c
(*env)->PushLocalFrame(env, 32);
// allocate up to 32 local refs
// ...
(*env)->PopLocalFrame(env, NULL); // frees them
```

### 4.8 Exercise Set (Practical)
1. Extend `NativeMath` with `subtract`.
2. Modify `ArrayOps` to return a new array instead of modifying.
3. Add exception throwing to `Counter` if counter > 100.
4. Implement thread attach and callback scenario.
5. Use `PushLocalFrame` for batch object creation.

---
<div style="page-break-before: always;"></div>

# Module 5 – Compilation & Runtime Errors

### 5.1 Signature Mismatch
Java: `public native long add(int a,int b);` vs C: returns `jint` -> runtime UnsatisfiedLinkError.

### 5.2 Library Not Found
`UnsatisfiedLinkError: no nativemath in java.library.path` – ensure library in path or set `-Djava.library.path`.

### 5.3 Forgetting to Load Library
Missing static block load -> native method unresolved.

### 5.4 Field Signature Wrong
Request field ID with `"J"` but field is `int` -> returns NULL; later use causes crash.

### 5.5 Not Releasing String/Array
Memory leak, increased GC pauses.

### 5.6 Invalid UTF Data
Incorrect encoding leading to lost characters; must use correct API.

### 5.7 Attaching Threads Twice
Repeated `AttachCurrentThread` may leak; detach when done.

### 5.8 Exercise Set (Errors)
Identify and fix: wrong return type, library path misconfiguration, field type mismatch.

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques & Hacks

### 6.1 Minimize Crossing Boundary
Aggregate operations to reduce overhead.

### 6.2 Precompute IDs
Cache `jclass`, `jmethodID`, `jfieldID` in `JNI_OnLoad`.

### 6.3 Use Critical Sections
`GetPrimitiveArrayCritical` for large arrays; release ASAP. Avoid blocking while pinned.

### 6.4 Direct ByteBuffers
Allocate direct buffers for zero-copy native I/O.

### 6.5 Error Translation Layer
Map native error codes to rich Java exceptions.

### 6.6 Safe Wrapper Abstraction
Expose high-level Java APIs hiding JNI complexity.

### 6.7 Testing Native Code
Use JUnit with static initialization + mock data; verify edge cases.

### 6.8 Build Automation
Use CMake or Gradle with `cpp-library` plugin; produce multi-platform artifacts.

### 6.9 Security Hardening
Fortify source, run static analysis (clang-tidy), memory sanitizer.

### 6.10 Exercise Set (Techniques)
Implement caching of method IDs and compare performance before/after.

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions

1. Explain JNI lifecycle from declaration to invocation.
2. Difference between local and global references.
3. Why minimize JNI calls?
4. How to handle exceptions in native code?
5. Role of `JNI_OnLoad`.
6. Memory management pitfalls using arrays.
7. Attaching threads – why and how.
8. When to use direct byte buffers.
9. Strategies to debug UnsatisfiedLinkError.
10. Security considerations when using JNI.

### Exercise
Explain how you would design a wrapper to a C image processing library minimizing JNI overhead.

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. Java class with `multiply` native method, header generated, implement in C. 2. Use `javac -h native`. 3. C implementation returning product. 4. Load library and invoke.

## Solutions: Module 2 Exercises
Field access using `GetFieldID`/`SetIntField`. Callback with `CallVoidMethod`. Object creation via `FindClass` + `NewObject`. `JNI_OnLoad` caches IDs. Thread attach via `AttachCurrentThread` then detach.

## Solutions: Module 3 MCQ Extension
16: Use `NewObject` to construct and return. 17: Cache in `JNI_OnLoad` for efficiency.

## Solutions: Module 4 Practical
Subtract: implement native and header signature. Return new array: create `NewIntArray` and set region. Exception if counter >100 using `ThrowNew`. Thread attach demo using background pthread. Batch object creation with `PushLocalFrame`.

## Solutions: Module 5 Errors
Correct return type to `jlong`, ensure library path via `-Djava.library.path=...`, correct field signature string `"I"`.

## Solutions: Module 6 Techniques
Benchmark caching vs non-caching; method IDs looked up once reduce overhead.

## Solutions: Module 7 Exercise
Design: One native call passing bulk pixel buffer, use direct byte buffer, perform all operations inside C, callback only for final result status.

---

### End Notes
JNI remains powerful but should be applied surgically: focus on minimizing boundary crossings, robust error handling, and strong memory discipline. Consider newer alternatives like Panama (Foreign Function & Memory API) for future evolution.
