# C# & .NET — Advanced Interview Topics

## LINQ — Language-Integrated Query

LINQ lets you write declarative data pipelines. Interviewers expect fluency — not just `Where` and `Select`, but understanding deferred execution, materialisation cost, and choosing the right operator.

### Deferred vs Immediate Execution

```csharp
// Deferred — nothing happens until you iterate
IEnumerable<int> evens = numbers.Where(n => n % 2 == 0); // no work yet
foreach (int n in evens) { }     // executes here

// Immediate — materialises right away
List<int> evenList = numbers.Where(n => n % 2 == 0).ToList(); // runs now
int count = numbers.Count(n => n % 2 == 0);                   // runs now
```

> **Interview trap:** chaining multiple enumerations on the same deferred query re-evaluates it each time. Call `.ToList()` when you need to iterate more than once.

### Powerful Operators Beyond the Basics

```csharp
// GroupBy — group elements by key
var grouped = words.GroupBy(w => w.Length);
// { 3: ["cat","dog"], 5: ["house","plane"] }

// Zip — pair two sequences element-wise
var pairs = names.Zip(scores, (name, score) => $"{name}: {score}");

// Aggregate — custom fold / reduce
int product = numbers.Aggregate(1, (acc, n) => acc * n);

// SelectMany — flatten nested collections
var allTags = posts.SelectMany(p => p.Tags);

// Chunk (C# 13 / .NET 9) — split into fixed-size batches
var batches = items.Chunk(100);
```

### Dictionary One-Liners

```csharp
// Frequency map — classic interview pattern
var freq = s.GroupBy(c => c).ToDictionary(g => g.Key, g => g.Count());

// Invert a dictionary
var inverted = dict.ToDictionary(kv => kv.Value, kv => kv.Key);

// Lookup (one-to-many) — safer than Dictionary<K, List<V>>
ILookup<string, Student> byCity = students.ToLookup(s => s.City);
```

---

## Async / Await

### How It Actually Works

`async`/`await` doesn't create new threads. It uses a **state machine** that suspends at each `await` and resumes when the awaited task completes, freeing the calling thread.

```csharp
// Correct — concurrent, then await all
async Task<int[]> FetchAllAsync(string[] urls)
{
    var tasks = urls.Select(url => httpClient.GetStringAsync(url)
                         .ContinueWith(t => t.Result.Length));
    return await Task.WhenAll(tasks);
}
```

### Common Pitfalls

```csharp
// ❌ Sequential by accident — each await blocks before starting the next
foreach (var url in urls)
    results.Add(await FetchAsync(url));

// ✅ Start all, then await all
var tasks = urls.Select(url => FetchAsync(url)).ToList();
var results = await Task.WhenAll(tasks);

// ❌ async void — exceptions are unobservable; only use for event handlers
async void BadMethod() { await Task.Delay(1000); }

// ✅ async Task
async Task GoodMethod() { await Task.Delay(1000); }

// ❌ .Result / .Wait() — can deadlock on UI/ASP.NET synchronisation context
var data = FetchAsync().Result; // deadlock risk

// ✅ await all the way up
var data = await FetchAsync();
```

### ValueTask — Avoid Heap Allocation

When a method often completes synchronously (e.g. cache hit), `ValueTask<T>` avoids allocating a `Task` object:

```csharp
async ValueTask<int> GetCachedValueAsync(string key)
{
    if (cache.TryGetValue(key, out int value))
        return value;                       // no allocation — synchronous path
    return await FetchFromDbAsync(key);      // async fallback
}
```

---

## Pattern Matching (C# 8–12)

Pattern matching goes way beyond `is` checks. Interviewers love seeing you use it to write concise, readable branching.

```csharp
// Type patterns
if (shape is Circle c)
    Console.WriteLine($"Radius: {c.Radius}");

// Property patterns
if (person is { Age: >= 18, Country: "US" })
    Console.WriteLine("Eligible");

// Switch expression (C# 8+)
string category = score switch
{
    >= 90 => "A",
    >= 80 => "B",
    >= 70 => "C",
    _     => "F"
};

// Tuple pattern — elegant state machines
string RockPaperScissors(string p1, string p2) => (p1, p2) switch
{
    ("rock", "scissors") => "P1 wins",
    ("scissors", "paper") => "P1 wins",
    ("paper", "rock")     => "P1 wins",
    _ when p1 == p2       => "Tie",
    _                     => "P2 wins"
};

// List patterns (C# 11+)
int[] arr = { 1, 2, 3, 4, 5 };
if (arr is [1, 2, .. var rest])
    Console.WriteLine(rest.Length); // 3

// Relational + logical patterns
string Classify(double bmi) => bmi switch
{
    < 18.5              => "Underweight",
    >= 18.5 and < 25.0  => "Normal",
    >= 25.0 and < 30.0  => "Overweight",
    _                   => "Obese"
};
```

---

## Span&lt;T&gt; and Memory&lt;T&gt;

Zero-allocation slicing for high-performance code. Interviewers at systems-focused companies ask about these.

```csharp
// Slice a string without allocating a new string
ReadOnlySpan<char> span = "Hello, World!".AsSpan();
ReadOnlySpan<char> hello = span[..5];    // "Hello" — no allocation

// Slice an array without copying
Span<int> slice = arr.AsSpan(2, 4);      // view into arr[2..6]
slice[0] = 99;                           // modifies arr[2] directly

// stackalloc — allocate small buffers on the stack
Span<int> buffer = stackalloc int[128];
buffer[0] = 42;
```

**Key rules:**
- `Span<T>` is stack-only (can't store in fields, can't use in async methods)
- `Memory<T>` is the heap-friendly sibling (can be stored, passed to async code)
- Both avoid GC pressure in hot paths

---

## Records and Value Semantics

```csharp
// Record — immutable reference type with value equality (C# 9+)
public record Point(int X, int Y);

var p1 = new Point(1, 2);
var p2 = new Point(1, 2);
Console.WriteLine(p1 == p2);        // true (value equality, not reference)
Console.WriteLine(p1);              // Point { X = 1, Y = 2 } (built-in ToString)

// Non-destructive mutation with `with`
var p3 = p1 with { X = 10 };       // Point { X = 10, Y = 2 }

// Record struct — value type with value equality (C# 10+)
public record struct Coord(double Lat, double Lon);
```

**When to use:**
- DTOs, config objects, domain events, immutable keys
- Anytime you want value equality without writing `Equals`/`GetHashCode`

---

## Delegates, Events, and Func/Action

```csharp
// Func<T, TResult> — takes input, returns output
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(3, 4)); // 7

// Action<T> — takes input, returns void
Action<string> log = msg => Console.WriteLine($"[LOG] {msg}");

// Predicate<T> — Func<T, bool> shorthand
Predicate<int> isEven = n => n % 2 == 0;
List<int> evens = numbers.FindAll(isEven);

// Multicast delegates — chain multiple handlers
Action<string> pipeline = msg => Console.Write("[A] ");
pipeline += msg => Console.Write("[B] ");
pipeline += msg => Console.WriteLine(msg);
pipeline("hello"); // [A] [B] hello
```

---

## Collections Interview Tricks

```csharp
// HashSet — O(1) lookups, set operations
var set = new HashSet<int>(listA);
set.IntersectWith(listB);          // in-place intersection
set.UnionWith(listC);              // in-place union
set.ExceptWith(listD);             // in-place difference

// SortedSet / SortedDictionary — O(log n) operations, ordered
var sorted = new SortedSet<int> { 5, 3, 1, 4 };
Console.WriteLine(sorted.Min);     // 1 — O(log n)

// PriorityQueue (C# 10+ / .NET 6+) — native min-heap
var pq = new PriorityQueue<string, int>();
pq.Enqueue("low", 3);
pq.Enqueue("high", 1);
Console.WriteLine(pq.Dequeue());   // "high" (priority 1)

// Frozen collections (.NET 8+) — immutable + optimised for reads
FrozenDictionary<string, int> lookup = dict.ToFrozenDictionary();

// TryGetValue — single lookup (avoid ContainsKey + indexer)
if (dict.TryGetValue(key, out var value))
    Console.WriteLine(value);

// GetValueOrDefault — concise read with fallback
int count = freq.GetValueOrDefault(ch, 0);
```

---

## String Performance

```csharp
// StringBuilder — O(n) string building vs O(n²) with concatenation
var sb = new StringBuilder();
for (int i = 0; i < 10_000; i++)
    sb.Append(i).Append(',');
string result = sb.ToString();

// String.Create — allocate and fill in one step (advanced)
string stars = string.Create(5, '*', (span, c) =>
{
    for (int i = 0; i < span.Length; i++) span[i] = c;
});

// Span-based parsing — zero-allocation number parsing
ReadOnlySpan<char> input = "12345".AsSpan();
int value = int.Parse(input);
```

---

## Modern C# One-Liners Interviewers Like

```csharp
// Deconstruct and swap
(a, b) = (b, a);

// Range and Index operators
int[] last3 = arr[^3..];       // last 3 elements
int lastItem = arr[^1];        // last element

// Null-coalescing assignment
cache ??= LoadExpensiveDefault();

// Target-typed new
Dictionary<string, List<int>> map = new();

// Global usings + file-scoped namespace (C# 10)
global using System.Collections.Generic;
namespace MyApp;                // no braces needed
```

---

## Key Interview Questions

| Question | Key Point |
|----------|-----------|
| Value type vs reference type? | struct on stack, class on heap; structs are copied, classes are referenced |
| What is boxing? | Wrapping a value type in an `object` — allocates on the heap |
| IEnumerable vs ICollection vs IList? | Enumerable = iterate; Collection = count + add; List = index access |
| What is the GC and its generations? | Gen 0 (short-lived), Gen 1 (buffer), Gen 2 (long-lived); Mark-and-sweep |
| Sealed class vs static class? | Sealed = can't inherit; Static = can't instantiate, all members static |
| Task vs Thread? | Task is a higher-level abstraction; uses thread pool, supports await |
| Why is string immutable? | Thread safety, interning, hash code caching |
