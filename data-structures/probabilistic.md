# Probabilistic & Approximate Data Structures

## Overview

These structures trade **exactness for massive space savings**. They answer questions like "have I seen this?", "how many unique items?", and "how frequent is this?" using sub-linear memory.

| Structure       | Question it Answers         | False Positives? | False Negatives? | Space      |
|----------------|-----------------------------|------------------|------------------|------------|
| Bloom Filter    | "Is X in the set?"          | Yes              | No               | O(m) bits  |
| HyperLogLog     | "How many unique items?"    | ≈ (2% error)     | —                | ~12 KB     |
| Count-Min Sketch| "How frequent is X?"        | Over-counts      | Never            | O(w × d)   |
| Skip List       | "Find/insert/delete X?"     | N/A (exact)      | N/A (exact)      | O(n) avg   |

---

## Bloom Filter

A space-efficient set membership test. Can say **"definitely not in the set"** or **"probably in the set"** — but never gives a false negative.

### How It Works
1. Create a bit array of size `m`, all zeros
2. Use `k` independent hash functions
3. **Insert:** hash the item `k` times, set those bit positions to 1
4. **Query:** hash the item `k` times; if ALL bits are 1, return "probably yes"; if ANY bit is 0, return "definitely no"

```javascript
class BloomFilter {
  constructor(size, numHashes) {
    this.bits = new Uint8Array(size);
    this.size = size;
    this.numHashes = numHashes;
  }

  _hashes(item) {
    const results = [];
    for (let i = 0; i < this.numHashes; i++) {
      // simplified: use different seeds
      let hash = 0;
      for (const ch of `${i}${item}`) hash = (hash * 31 + ch.charCodeAt(0)) >>> 0;
      results.push(hash % this.size);
    }
    return results;
  }

  add(item) {
    for (const h of this._hashes(item)) this.bits[h] = 1;
  }

  mightContain(item) {
    return this._hashes(item).every(h => this.bits[h] === 1);
  }
}

const bf = new BloomFilter(1000, 3);
bf.add("hello");
console.log(bf.mightContain("hello")); // true
console.log(bf.mightContain("world")); // false (probably)
```

```python
class BloomFilter:
    def __init__(self, size, num_hashes):
        self.bits = [0] * size
        self.size = size
        self.num_hashes = num_hashes

    def _hashes(self, item):
        results = []
        for i in range(self.num_hashes):
            h = hash(f"{i}{item}") % self.size
            results.append(h)
        return results

    def add(self, item):
        for h in self._hashes(item):
            self.bits[h] = 1

    def might_contain(self, item):
        return all(self.bits[h] for h in self._hashes(item))

bf = BloomFilter(1000, 3)
bf.add("hello")
print(bf.might_contain("hello"))  # True
print(bf.might_contain("world"))  # False (probably)
```

```csharp
public class BloomFilter
{
    private readonly bool[] bits;
    private readonly int numHashes;

    public BloomFilter(int size, int numHashes)
    {
        bits = new bool[size];
        this.numHashes = numHashes;
    }

    private int[] GetHashes(string item)
    {
        var results = new int[numHashes];
        for (int i = 0; i < numHashes; i++)
            results[i] = Math.Abs($"{i}{item}".GetHashCode()) % bits.Length;
        return results;
    }

    public void Add(string item)
    {
        foreach (int h in GetHashes(item)) bits[h] = true;
    }

    public bool MightContain(string item) =>
        GetHashes(item).All(h => bits[h]);
}
```

**Real-world uses:** Google Chrome (malicious URL check), databases (avoid unnecessary disk reads), Redis, Cassandra.

---

## HyperLogLog

Estimates the count of **unique elements** in a massive dataset using ~12 KB of memory regardless of how many items you process.

### Key Idea
Hash each item and count the maximum number of leading zeros seen. Statistically, if the longest run of leading zeros is `p`, there are approximately 2^p unique items. HyperLogLog improves accuracy by using many "registers" (buckets) and taking a harmonic mean.

| Property         | Value                            |
|-----------------|----------------------------------|
| Space            | ~12 KB (for 2^14 registers)      |
| Error rate       | ~0.81% standard error            |
| Operations       | Add, Count (no remove)           |
| Used by          | Redis `PFADD`/`PFCOUNT`, BigQuery |

```python
# Conceptual — real implementations use murmur hash + register math
import hashlib

class SimpleHyperLogLog:
    def __init__(self, num_buckets=64):
        self.num_buckets = num_buckets
        self.registers = [0] * num_buckets

    def add(self, item):
        h = int(hashlib.md5(str(item).encode()).hexdigest(), 16)
        bucket = h % self.num_buckets
        remaining = h // self.num_buckets
        # count leading zeros
        zeros = 0
        while remaining > 0 and remaining % 2 == 0:
            zeros += 1
            remaining //= 2
        self.registers[bucket] = max(self.registers[bucket], zeros + 1)

    def count(self):
        # harmonic mean estimate
        alpha = 0.7213 / (1 + 1.079 / self.num_buckets)
        raw = alpha * self.num_buckets ** 2
        raw /= sum(2 ** (-r) for r in self.registers)
        return int(raw)
```

---

## Count-Min Sketch

Estimates the **frequency** of items in a stream. Uses a 2D grid of counters with multiple hash functions. Always over-counts, never under-counts.

### How It Works
1. Create a `d × w` grid of counters (d = number of hash functions, w = width)
2. **Insert:** hash item with each of d functions, increment counter at each position
3. **Query:** return the **minimum** across all d counters for that item

```javascript
class CountMinSketch {
  constructor(width, depth) {
    this.width = width;
    this.depth = depth;
    this.table = Array.from({length: depth}, () => new Array(width).fill(0));
  }

  _hash(item, seed) {
    let h = seed;
    for (const ch of String(item)) h = (h * 31 + ch.charCodeAt(0)) >>> 0;
    return h % this.width;
  }

  add(item, count = 1) {
    for (let i = 0; i < this.depth; i++)
      this.table[i][this._hash(item, i + 1)] += count;
  }

  estimate(item) {
    let min = Infinity;
    for (let i = 0; i < this.depth; i++)
      min = Math.min(min, this.table[i][this._hash(item, i + 1)]);
    return min;
  }
}
```

```python
class CountMinSketch:
    def __init__(self, width, depth):
        self.width = width
        self.depth = depth
        self.table = [[0] * width for _ in range(depth)]

    def _hash(self, item, seed):
        return hash(f"{seed}{item}") % self.width

    def add(self, item, count=1):
        for i in range(self.depth):
            self.table[i][self._hash(item, i)] += count

    def estimate(self, item):
        return min(self.table[i][self._hash(item, i)] for i in range(self.depth))
```

```csharp
public class CountMinSketch
{
    private readonly int[,] table;
    private readonly int width, depth;

    public CountMinSketch(int width, int depth)
    {
        this.width = width;
        this.depth = depth;
        table = new int[depth, width];
    }

    private int Hash(string item, int seed) =>
        Math.Abs($"{seed}{item}".GetHashCode()) % width;

    public void Add(string item, int count = 1)
    {
        for (int i = 0; i < depth; i++)
            table[i, Hash(item, i)] += count;
    }

    public int Estimate(string item)
    {
        int min = int.MaxValue;
        for (int i = 0; i < depth; i++)
            min = Math.Min(min, table[i, Hash(item, i)]);
        return min;
    }
}
```

**Real-world uses:** Network traffic monitoring, trending topics, rate limiting, database query optimization.

---

## Skip List

A layered linked list that achieves **O(log n) search** on average by adding "express lane" pointers at multiple levels. Used in Redis sorted sets.

### How It Works
- Bottom layer: regular sorted linked list
- Each higher layer: random subset of nodes from the layer below (each node is promoted with probability 1/2)
- Search: start at top layer, move right until you overshoot, drop down one layer, repeat

| Operation | Average  | Worst    |
|----------|----------|----------|
| Search    | O(log n) | O(n)     |
| Insert    | O(log n) | O(n)     |
| Delete    | O(log n) | O(n)     |
| Space     | O(n)     | O(n log n)|

```python
import random

class SkipNode:
    def __init__(self, val, level):
        self.val = val
        self.forward = [None] * (level + 1)

class SkipList:
    def __init__(self, max_level=16):
        self.max_level = max_level
        self.level = 0
        self.header = SkipNode(-1, max_level)

    def _random_level(self):
        lvl = 0
        while random.random() < 0.5 and lvl < self.max_level:
            lvl += 1
        return lvl

    def search(self, target):
        current = self.header
        for i in range(self.level, -1, -1):
            while current.forward[i] and current.forward[i].val < target:
                current = current.forward[i]
        current = current.forward[0]
        return current is not None and current.val == target

    def insert(self, val):
        update = [None] * (self.max_level + 1)
        current = self.header
        for i in range(self.level, -1, -1):
            while current.forward[i] and current.forward[i].val < val:
                current = current.forward[i]
            update[i] = current

        new_level = self._random_level()
        if new_level > self.level:
            for i in range(self.level + 1, new_level + 1):
                update[i] = self.header
            self.level = new_level

        new_node = SkipNode(val, new_level)
        for i in range(new_level + 1):
            new_node.forward[i] = update[i].forward[i]
            update[i].forward[i] = new_node
```

---

## When to Use What

| Need                                  | Structure          |
|--------------------------------------|--------------------|
| "Have I seen this before?" (set test) | Bloom Filter       |
| Count unique visitors / items         | HyperLogLog        |
| Track frequency in a stream           | Count-Min Sketch   |
| Fast sorted operations (Redis-like)   | Skip List          |
