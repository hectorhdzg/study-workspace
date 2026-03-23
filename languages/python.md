# Python — Advanced Interview Topics

## Generators and Iterators

Generators produce values **lazily** — one at a time — instead of building the entire collection in memory. Interviewers love asking about `yield`, `StopIteration`, and memory efficiency.

### Generator Functions

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a          # pauses here, resumes on next()
        a, b = b, a + b

gen = fibonacci()
print([next(gen) for _ in range(8)])  # [0, 1, 1, 2, 3, 5, 8, 13]
```

### Generator Expressions (Lazy comprehensions)

```python
# List comprehension — builds entire list in memory
squares_list = [x**2 for x in range(1_000_000)]     # ~8 MB

# Generator expression — produces one at a time
squares_gen = (x**2 for x in range(1_000_000))       # ~120 bytes
print(sum(squares_gen))  # consumes lazily
```

### yield from — Delegate to Sub-generators

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)   # recursive delegation
        else:
            yield item

print(list(flatten([1, [2, [3, 4]], 5])))  # [1, 2, 3, 4, 5]
```

### The Iterator Protocol

Any object with `__iter__` and `__next__` is an iterator. Generators implement this automatically.

```python
class CountDown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        return self

    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

list(CountDown(3))  # [3, 2, 1]
```

---

## Decorators

Functions that **wrap other functions** to add behavior. Ubiquitous in Python interviews.

### Basic Decorator Pattern

```python
import functools
import time

def timer(func):
    @functools.wraps(func)           # preserves __name__, __doc__
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.5)
    return "done"
```

### Decorator with Arguments

```python
def retry(max_attempts=3):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts:
                        raise
                    print(f"Attempt {attempt} failed: {e}")
        return wrapper
    return decorator

@retry(max_attempts=5)
def flaky_api_call():
    ...
```

### Memoization Decorator (Interview Classic)

```python
def memoize(func):
    cache = {}
    @functools.wraps(func)
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fib(n):
    if n < 2: return n
    return fib(n - 1) + fib(n - 2)

# Or just use the built-in:
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    if n < 2: return n
    return fib(n - 1) + fib(n - 2)
```

---

## Context Managers

The `with` statement guarantees **cleanup runs** even if an exception occurs. Interviewers ask you to write custom ones.

### Using `__enter__` / `__exit__`

```python
class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.4f}s")
        return False   # don't suppress exceptions

with Timer() as t:
    time.sleep(1)
# prints: Elapsed: 1.000Xs
```

### Using `contextlib` (Simpler)

```python
from contextlib import contextmanager

@contextmanager
def temp_directory():
    import tempfile, shutil
    path = tempfile.mkdtemp()
    try:
        yield path           # code inside `with` runs here
    finally:
        shutil.rmtree(path)  # cleanup always runs

with temp_directory() as d:
    print(f"Working in {d}")
```

---

## Dunder (Magic) Methods

These define how objects behave with operators, `len()`, `str()`, comparisons, etc. A common interview topic.

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

    def __add__(self, other):          # v1 + v2
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other):           # v1 == v2
        return self.x == other.x and self.y == other.y

    def __hash__(self):                # needed if __eq__ is defined
        return hash((self.x, self.y))

    def __len__(self):                 # len(v)
        return int((self.x**2 + self.y**2) ** 0.5)

    def __getitem__(self, index):      # v[0], v[1]
        return (self.x, self.y)[index]

    def __iter__(self):                # for component in v
        yield self.x
        yield self.y

    def __contains__(self, value):     # value in v
        return value in (self.x, self.y)

v1 = Vector(1, 2)
v2 = Vector(3, 4)
print(v1 + v2)        # Vector(4, 6)
print(v1 == Vector(1, 2))  # True
print(list(v1))        # [1, 2]
```

---

## The GIL (Global Interpreter Lock)

The GIL ensures only **one thread executes Python bytecode at a time** in CPython. This is the #1 Python concurrency interview topic.

| Scenario | GIL Impact | Solution |
|----------|-----------|----------|
| CPU-bound (math, parsing) | GIL blocks parallelism | Use `multiprocessing` or C extensions |
| I/O-bound (network, files) | GIL is released during I/O | Use `threading` or `asyncio` |
| Data science / numpy | Numpy releases GIL for C ops | Threading works fine |

```python
# CPU-bound — use multiprocessing to bypass GIL
from multiprocessing import Pool

def heavy_computation(n):
    return sum(i * i for i in range(n))

with Pool(4) as p:
    results = p.map(heavy_computation, [10**6] * 4)

# I/O-bound — use asyncio
import asyncio, aiohttp

async def fetch(session, url):
    async with session.get(url) as resp:
        return await resp.text()

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        return await asyncio.gather(*(fetch(session, u) for u in urls))
```

---

## Collections Module — Hidden Gems

```python
from collections import defaultdict, Counter, deque, OrderedDict, namedtuple

# defaultdict — auto-initialises missing keys
graph = defaultdict(list)
graph["A"].append("B")         # no KeyError

# Counter — frequency counting in one line
freq = Counter("abracadabra")  # Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
freq.most_common(2)            # [('a', 5), ('b', 2)]
counter1 + counter2            # add counts
counter1 & counter2            # min of each (intersection)

# deque — O(1) append/pop on BOTH ends (list is O(n) for popleft)
q = deque([1, 2, 3])
q.appendleft(0)                # [0, 1, 2, 3]
q.rotate(1)                    # [3, 0, 1, 2]

# namedtuple — lightweight immutable data
Point = namedtuple("Point", ["x", "y"])
p = Point(1, 2)
print(p.x, p.y)               # 1 2
```

---

## Comprehensions — Beyond the Basics

```python
# Dict comprehension — frequency map
freq = {c: s.count(c) for c in set(s)}

# Set comprehension — unique lengths
lengths = {len(w) for w in words}

# Nested comprehension — flatten
flat = [x for row in matrix for x in row]

# Conditional comprehension
positives = [x for x in nums if x > 0]

# Walrus operator in comprehension (3.8+)
results = [y for x in data if (y := expensive(x)) > threshold]
```

---

## Dataclasses (3.7+) and Slots

```python
from dataclasses import dataclass, field

@dataclass(frozen=True)              # frozen = immutable + hashable
class Point:
    x: float
    y: float

@dataclass
class Student:
    name: str
    grades: list[int] = field(default_factory=list)
    gpa: float = field(init=False)   # computed, not in __init__

    def __post_init__(self):
        self.gpa = sum(self.grades) / len(self.grades) if self.grades else 0.0

# __slots__ — restrict attributes, save memory, faster access
@dataclass(slots=True)    # Python 3.10+
class Pixel:
    x: int
    y: int
    color: str
```

---

## Useful Built-in Functions

```python
# zip — pair up iterables (stops at shortest)
for name, score in zip(names, scores):
    print(f"{name}: {score}")

# zip with strict (3.10+) — raises if lengths differ
list(zip(a, b, strict=True))

# enumerate — index + value
for i, item in enumerate(items, start=1):
    print(f"{i}. {item}")

# any / all — short circuit boolean checks
if any(x > 100 for x in values): ...
if all(x > 0 for x in values): ...

# sorted with key — custom sort
sorted(students, key=lambda s: (-s.gpa, s.name))

# itertools — common interview patterns
from itertools import chain, groupby, product, combinations, permutations, accumulate

# chain — flatten multiple iterables
list(chain([1,2], [3,4], [5]))  # [1, 2, 3, 4, 5]

# accumulate — running totals (prefix sum)
list(accumulate([1, 2, 3, 4]))  # [1, 3, 6, 10]

# groupby — group consecutive equal elements (sort first for full grouping)
from itertools import groupby
data = sorted(items, key=lambda x: x.category)
for key, group in groupby(data, key=lambda x: x.category):
    print(key, list(group))
```

---

## Key Interview Questions

| Question | Key Point |
|----------|-----------|
| List vs tuple? | List is mutable; tuple is immutable, hashable, slightly faster |
| `is` vs `==`? | `is` checks identity (same object); `==` checks value equality |
| Shallow vs deep copy? | Shallow copies references; deep copy recursively creates new objects |
| What is `*args, **kwargs`? | `*args` = variable positional args (tuple); `**kwargs` = variable keyword args (dict) |
| How does dict maintain order? | Insertion-ordered since Python 3.7 (implementation detail in 3.6) |
| What is a descriptor? | Object with `__get__`, `__set__`, `__delete__` — how `property()` works |
| `__new__` vs `__init__`? | `__new__` creates the instance; `__init__` initialises it |
| Global vs local scope? | LEGB rule: Local → Enclosing → Global → Built-in |
