# Java Date & Time API (Java 8, 17 & 20) – In-Depth Training

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

### 1.1 Why a New API?
The legacy `java.util.Date` & `Calendar` APIs were mutable, confusing (zero-based months), not thread-safe, and poorly designed for internationalization. Java 8 introduced the modern, immutable, thread-safe Date-Time API under `java.time` influenced by Joda-Time.

### 1.2 Core Principles
- Immutability: All primary temporal types are immutable & thread-safe.
- Clarity: Explicit separation of machine time (`Instant`) and human calendar representations (`LocalDate`, `LocalDateTime`).
- Domain separation: Date-only vs Time-only vs Zone-aware vs Offset-aware.
- Adjusters & amount types (`Period`, `Duration`) distinct.

### 1.3 Primary Classes Overview
| Category | Classes | Description |
|----------|---------|-------------|
| Date | `LocalDate` | Date without time-zone |
| Time | `LocalTime` | Time without date or zone |
| Date-Time | `LocalDateTime` | Date & time w/o zone |
| Instant | `Instant` | Machine timestamp (UTC) |
| Zone-aware | `ZonedDateTime` | Date-time with zone rules |
| Offset | `OffsetDateTime` / `OffsetTime` | With fixed offset from UTC |
| Duration/Period | `Duration`, `Period` | Time-based vs date-based amount |
| Formatter | `DateTimeFormatter` | Formatting/parsing patterns |
| Zone | `ZoneId`, `ZoneOffset` | Time zone region & offset |
| Utilities | `TemporalAdjuster`, `TemporalQuery` | Rule-based adjustments & queries |

### 1.4 ASCII Layer Diagram
```
+-----------------------------+
|         High-Level          | --> LocalDate, LocalTime, LocalDateTime
+-----------------------------+
|   Zone / Offset Layer       | --> ZonedDateTime, OffsetDateTime
+-----------------------------+
|    Instant (UTC Timeline)   | --> Instant
+-----------------------------+
|    Amount Types             | --> Period (date), Duration (time)
+-----------------------------+
|    Formatting & Parsing     | --> DateTimeFormatter
+-----------------------------+
```

### 1.5 Obtaining Current Values
```java
LocalDate today = LocalDate.now();
LocalTime nowTime = LocalTime.now();
LocalDateTime nowDT = LocalDateTime.now();
Instant instant = Instant.now();
ZonedDateTime zoned = ZonedDateTime.now(ZoneId.of("Europe/Paris"));
```

### 1.6 Construction
```java
LocalDate date = LocalDate.of(2025, Month.OCTOBER, 28);
LocalTime time = LocalTime.of(14, 30, 0);
LocalDateTime dt = LocalDateTime.of(date, time);
ZonedDateTime zdt = dt.atZone(ZoneId.of("UTC"));
Instant fromEpoch = Instant.ofEpochSecond(1_000_000L);
```

### 1.7 Parsing & Formatting
```java
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
LocalDateTime parsed = LocalDateTime.parse("2025-10-28 16:45", fmt);
String out = fmt.format(parsed);
```

### 1.8 Converting Between Representations
```java
Instant now = Instant.now();
ZonedDateTime paris = now.atZone(ZoneId.of("Europe/Paris"));
LocalDate dateOnly = paris.toLocalDate();
OffsetDateTime offset = paris.toOffsetDateTime();
```

### 1.9 Period vs Duration
- `Period`: date-based (years, months, days) respects calendar variations.
- `Duration`: time-based (seconds, nanos) precise machine measurement.

```java
Period p = Period.between(LocalDate.of(2024,1,1), LocalDate.of(2025,10,28));
Duration d = Duration.between(Instant.ofEpochSecond(0), Instant.now());
```

### 1.10 Arithmetic & Adjusters
```java
LocalDate nextMonday = LocalDate.now().with(TemporalAdjusters.next(DayOfWeek.MONDAY));
LocalDate plus90 = LocalDate.now().plusDays(90);
ZonedDateTime meeting = ZonedDateTime.now().withHour(9).withMinute(0).withSecond(0);
```

### 1.11 Exercise Set (Basics)
1. Create a `LocalDate` for your next birthday.
2. Parse "2025-12-31 23:59" into `LocalDateTime`.
3. Convert current `Instant` to `ZonedDateTime` in Tokyo.
4. Compute days until end of year.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 2 – In-Depth Topics

### 2.1 Time Zones & DST
Zones have historical rules (IANA database). `ZonedDateTime` applies transitions (e.g., DST gap/overlap). Ambiguous or invalid times may require `ZoneOffsetTransition` resolution.

DST Gap Example (Spring Forward): Some local times do not exist.

### 2.2 Handling Ambiguous Times
Use `ZonedDateTime.ofLocal(LocalDateTime, ZoneId, ZoneOffset preferredOffset)` or `LocalDateTime.atZone(zone)` with rules; overlapping times resolved to earlier offset by default.

### 2.3 Clock Abstraction
`Clock` allows injecting controllable time source (testing).
```java
Clock fixed = Clock.fixed(Instant.parse("2025-10-28T12:00:00Z"), ZoneOffset.UTC);
Instant testInstant = Instant.now(fixed);
```

### 2.4 Instant vs LocalDateTime
`Instant` is absolute moment; `LocalDateTime` is wall time without zone. Convert between using zone rules (`atZone`, `toInstant`).

### 2.5 Epoch Conversions
```java
long epochMilli = Instant.now().toEpochMilli();
Instant fromMilli = Instant.ofEpochMilli(epochMilli);
```

### 2.6 Temporal Queries
```java
LocalDate date = LocalDate.now();
TemporalQuery<Boolean> isLeap = temporal -> ((LocalDate) temporal).isLeapYear();
boolean leap = date.query(isLeap);
```

### 2.7 Advanced Formatting
Pattern symbols: `yyyy`, `MM`, `dd`, `HH`, `mm`, `ss`, `XXX` (zone offset). Use `LocalizedDateTimeFormatter` for locale-sensitive output.
```java
DateTimeFormatter iso = DateTimeFormatter.ISO_ZONED_DATE_TIME;
String s = iso.format(ZonedDateTime.now());
```

### 2.8 Duration & Period Arithmetic Edge Cases
Leap years & month length variations.

### 2.9 Comparing Temporal Types
```java
LocalDate a = LocalDate.of(2025,1,1); LocalDate b = LocalDate.of(2025,10,28);
boolean before = a.isBefore(b); boolean equal = a.isEqual(b);
```

### 2.10 Truncation & Rounding
```java
Instant truncated = Instant.now().truncatedTo(ChronoUnit.SECONDS);
ZonedDateTime hourStart = ZonedDateTime.now().truncatedTo(ChronoUnit.HOURS);
```

### 2.11 Performance Considerations
Immutable objects reduce need for synchronization. Parsing overhead can be mitigated by reusing `DateTimeFormatter` instances (they are thread-safe).

### 2.12 Zone Rules Caching
`ZoneId` queries cache internally; minimize custom timezone computations.

### 2.13 Java 17 & 20 Evolution
API stable since Java 8; Java 17/20 add pattern matching, records (enhance date/time domain modeling), improved time-source for tests; no major changes to `java.time` core but synergy with newer language features.

### 2.14 Exercise Set (In-Depth)
1. Resolve an ambiguous time during DST overlap.
2. Use `Clock.fixed` in a test scenario to compute elapsed duration.
3. Format a timestamp with offset and custom pattern.
4. Compute months between two dates using `Period`.

(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 3 – Multiple Choice Questions (MCQs)

| # | Question | A | B | C | D | Answer | Explanation |
|---|----------|---|---|---|---|--------|-------------|
| 1 | Which is immutable? | Date | Calendar | LocalDate | SimpleDateFormat | C | LocalDate is immutable; Date & Calendar mutable. |
| 2 | Represents an instantaneous point? | LocalDateTime | Instant | LocalDate | MonthDay | B | Instant is UTC timestamp. |
| 3 | Measures machine time? | Period | Duration | ZonedDateTime | YearMonth | B | Duration is time-based seconds/nanos. |
| 4 | Represents date without year? | YearMonth | MonthDay | LocalDate | LocalTime | B | MonthDay stores month & day only. |
| 5 | Formatting thread-safe class? | DateFormat | SimpleDateFormat | DateTimeFormatter | CalendarFormatter | C | DateTimeFormatter is thread-safe. |
| 6 | DST overlap resolution defaults to? | Later offset | Earlier offset | Throws exception | Random | B | Uses earlier offset by default. |
| 7 | Convert Instant to ZonedDateTime? | `instant.toZone()` | `ZonedDateTime.ofInstant(instant, zone)` | `instant.atZone(zone)` | both B and C | D | Both patterns valid. |
| 8 | `Period.between(a,b)` returns? | Partial duration ignoring months | Date-based difference | Time-based seconds | Offset value | B | Period is date-based. |
| 9 | Truncate Instant to minutes? | `instant.truncatedTo(ChronoUnit.MINUTES)` | `instant.toMinutes()` | `instant.truncate(MINUTES)` | `instant.floor(MINUTES)` | A | Correct API call. |
| 10 | Pattern symbol for 24h hour? | `hh` | `HH` | `KK` | `kk` | B | HH is 00-23. |
| 11 | Pattern for zone offset like +02:00? | `ZZZ` | `XXX` | `OOO` | `PPP` | B | XXX outputs ISO offset. |
| 12 | Add one month safely? | `date.plusMonths(1)` | `date.addMonth(1)` | `date.plus(1,MONTH)` | `date.add(MONTH,1)` | A | plusMonths exists. |
| 13 | Leap year check? | `date.isLeapYear()` | `Year.isLeap(date)` | `date.isLeap()` | `LeapYear.of(date)` | A | Method isLeapYear(). |
| 14 | Which is time-only? | LocalTime | OffsetDateTime | LocalDateTime | Instant | A | LocalTime has no date. |
| 15 | Thread-safe formatting strategy? | Create new SimpleDateFormat each time | Use shared DateTimeFormatter | Synchronize on Calendar | Use static DateFormat | B | DateTimeFormatter is thread-safe.

### MCQ Extension Exercise
Write 2 MCQs: one about `Clock`, one about DST gap.

---
<div style="page-break-before: always;"></div>

# Module 4 – Practical Hands-On Exercises

### Exercise 4.1: End-of-Month Billing Date
Compute billing date (last business day of month) ignoring weekends.

### Exercise 4.2: Countdown to Event
Given event date/time in zone, compute remaining days/hours.

### Exercise 4.3: DST Overlap Handling
Create LocalDateTime that falls in overlap; map to both offsets available.

### Exercise 4.4: Fixed Clock Testing
Use fixed clock to test time-dependent service output.

### Exercise 4.5: Period Summary
Produce human-readable difference between two dates (e.g., "1 year, 2 months, 5 days").

### Exercise 4.6: Truncation Utility
Write method to truncate ZonedDateTime to start of day in its zone.

### Exercise 4.7: Time Window Checker
Check if current time is within business hours 09:00–17:30 local zone.

### Exercise 4.8: Instant Serialization
Serialize Instant to ISO string and parse back.

---
<div style="page-break-before: always;"></div>

# Module 5 – Common Compilation & Runtime Errors

### 5.1 Using Month Index (Old API Habit)
```java
LocalDate wrong = LocalDate.of(2025, 0, 10); // Compile error (month starts at 1)
```

### 5.2 Mutable Formatter Assumption
Reusing `SimpleDateFormat` across threads leading to corrupted output (legacy API misuse). Solution: use `DateTimeFormatter`.

### 5.3 Parsing Pattern Mismatch
```java
LocalDate.parse("28/10/2025", DateTimeFormatter.ofPattern("yyyy-MM-dd")); // DateTimeParseException
```

### 5.4 Converting LocalDateTime To Instant Without Zone
```java
LocalDateTime ldt = LocalDateTime.now();
// ldt.toInstant(); // Compile error: need zone
```
Correct:
```java
Instant inst = ldt.atZone(ZoneId.of("UTC")).toInstant();
```

### 5.5 Wrong ChronoUnit
```java
Instant.now().plus(5, ChronoUnit.MONTHS); // UnsupportedTemporalTypeException
```

### 5.6 DST Gap Scheduling
Scheduling a non-existent time results in automatic adjustment (next valid time) or exception in some libraries.

### 5.7 Mixing Period & Duration incorrectly
Adding `Duration` to `LocalDate` fails (needs date-time or time). Use `date.atStartOfDay().plus(duration)`.

### 5.8 Exercise: Identify Errors
```java
LocalDate d = LocalDate.of(2025, 13, 1);
Duration dur = Period.ofDays(5).addTo(Duration.ZERO); // misuse
Instant inst = LocalDate.now().toInstant(ZoneOffset.UTC);
```
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 6 – Techniques, Patterns & Hacks

### 6.1 Business Calendar Utility
Use `TemporalAdjuster` to skip weekends.
```java
TemporalAdjuster lastBusinessDayOfMonth = temporal -> {
    LocalDate date = LocalDate.from(temporal).with(TemporalAdjusters.lastDayOfMonth());
    while(date.getDayOfWeek() == DayOfWeek.SATURDAY || date.getDayOfWeek() == DayOfWeek.SUNDAY){
        date = date.minusDays(1);
    }
    return date;
};
```

### 6.2 Human Duration Formatter
Aggregate `Period` & leftover `Duration` for events.

### 6.3 Reusable Formatters
```java
static final DateTimeFormatter BASIC_DATE = DateTimeFormatter.ofPattern("yyyyMMdd");
```

### 6.4 Caching ZoneIds
Store frequently used `ZoneId` constants to avoid repeated lookups.

### 6.5 Range Checks
```java
boolean inRange = !date.isBefore(start) && !date.isAfter(end);
```

### 6.6 Time Arithmetic Safe Wrapper
Provide utility that uses `plusMonths` carefully handling end-of-month rollover.

### 6.7 Using `Clock` for Dependency Injection
Pass `Clock` to services for testability.

### 6.8 Layered DTO Using Records
```java
record EventWindow(Instant start, Instant end) { boolean contains(Instant i){ return !i.isBefore(start) && !i.isAfter(end); } }
```

### 6.9 Precomputing Next Execution
Compute next schedule time using adjusters & zone transitions.

### 6.10 Exercise: Implement human-readable difference function returning "Y years M months D days".

---
<div style="page-break-before: always;"></div>

# Module 7 – Interview Questions & Answers

1. Difference between `Period` and `Duration`?  
Date vs time-based; Period variable length components, Duration fixed second/nano count.
2. Why is `DateTimeFormatter` thread-safe?  
Immutable design; internal state not mutated per format call.
3. How do you convert `LocalDateTime` to `Instant`?  
Attach zone: `ldt.atZone(zone).toInstant()`.
4. What is an ambiguous date-time?  
Occurs during DST fall-back when same local time maps to two offsets.
5. How to detect DST gap?  
Check `ZoneRules` transitions around target LocalDateTime.
6. Best way to handle month-end rollover?  
Use adjusters (e.g., add month then adjust to last valid day if needed).
7. Why prefer immutability for temporal types?  
Thread safety & easier reasoning.
8. When would you use `Clock`?  
Testing, simulation, multi-zone retrieval abstraction.
9. What is `Instant` good for?  
Timestamps for logging, persistence; machine time comparisons.
10. Why separate LocalDate and LocalTime?  
Clarify intent, avoid mixing semantics when only date or time needed.

### Exercise: Explain approach to formatting a ZonedDateTime with locale & offset.
(See Module 8.)

---
<div style="page-break-before: always;"></div>

# Module 8 – Solutions Appendix

## Solutions: Module 1 Exercises
1. `LocalDate nextBirthday = LocalDate.of(2026, Month.JULY, 5);` (Adjust for actual inputs.)
2. `LocalDateTime.parse("2025-12-31 23:59", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm"));`
3. `ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));`
4. `long days = ChronoUnit.DAYS.between(LocalDate.now(), LocalDate.of(LocalDate.now().getYear(),12,31));`

## Solutions: Module 2 Exercises
1. Ambiguous time resolution example with `ZonedDateTime.ofLocal` using preferred offset.
2. Fixed clock difference: measure two instants.
3. Custom pattern: `DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm XXX");`
4. Months between: `Period.between(start, end).getMonths();`

## Solutions: Module 3 MCQ Extension
1. Clock MCQ: `Clock.fixed` allows deterministic testing; answer revolve around controllable time source.
2. DST gap MCQ: Non-existent time auto-adjust or requires explicit rule handling.

## Solutions: Module 4 Selected Exercises
Exercise 4.1 adjuster implementation provided earlier.
Exercise 4.5 formatting difference example skeleton.
Exercise 4.6: `zdt.truncatedTo(ChronoUnit.DAYS);`
Exercise 4.7: Business hours check using LocalTime comparisons.
Exercise 4.8: `String iso = instant.toString(); Instant parsed = Instant.parse(iso);`

## Solutions: Module 5 Error Identification (5.8)
Errors:
1. Month 13 invalid: months 1-12.
2. Using Period to create Duration misuse: distinct types; building Duration from seconds or `Duration.ofDays(5)`.
3. `LocalDate` lacks direct `toInstant`; need `atStartOfDay(zone).toInstant()`.

## Solutions: Module 6 Human-readable Difference
Compute years, months, days via Period; format string accordingly.

## Solutions: Module 7 Exercise
Use `DateTimeFormatter.ofLocalizedDateTime(FormatStyle.FULL).withLocale(Locale.FRANCE)` then `format(zdt)` ensures locale-specific names; include offset with pattern extension.

---

### End Notes
Treat time data explicitly: choose correct abstraction (Instant vs LocalDateTime vs ZonedDateTime) and rely on immutable operations with adjusters for correctness across time zones and DST transitions.
