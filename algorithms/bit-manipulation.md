# Bit Manipulation

## Key Operations

| Operation         | Syntax (JS) | Example                         |
|------------------|-------------|----------------------------------|
| AND              | `a & b`     | `5 & 3 = 1` (101 & 011 = 001)  |
| OR               | `a \| b`    | `5 \| 3 = 7` (101 \| 011 = 111)|
| XOR              | `a ^ b`     | `5 ^ 3 = 6` (101 ^ 011 = 110)  |
| NOT              | `~a`        | `~5 = -6`                       |
| Left shift       | `a << n`    | `1 << 3 = 8`                    |
| Right shift      | `a >> n`    | `8 >> 2 = 2`                    |

## Common Tricks

Bit manipulation lets you solve certain problems in O(1) space and with very fast constant factors. The core insight: bitwise operations work on individual bits in parallel, making them ideal for toggling, checking, and counting properties of binary representations.

- **Check / Set / Clear / Toggle a bit** — Use masks with shifts. `1 << i` creates a mask with only bit `i` set.
- **Power of 2 check** — A power of 2 has exactly one bit set. `n & (n-1)` clears the lowest set bit — if the result is 0, there was only one bit.
- **Count set bits (Brian Kernighan)** — Repeatedly clear the lowest set bit with `n &= n-1` and count iterations. This runs in O(k) where k is the number of set bits, not the total bit width.
- **XOR to find the unique element** — XOR is self-canceling: `a ^ a = 0` and `a ^ 0 = a`. XOR all elements together — duplicates cancel out, leaving the single unique value.

```javascript
// Check if bit i is set
function isBitSet(num, i) {
  return (num & (1 << i)) !== 0;
}

// Set bit i
function setBit(num, i) {
  return num | (1 << i);
}

// Clear bit i
function clearBit(num, i) {
  return num & ~(1 << i);
}

// Toggle bit i
function toggleBit(num, i) {
  return num ^ (1 << i);
}

// Check if power of 2
function isPowerOfTwo(n) {
  return n > 0 && (n & (n - 1)) === 0;
}

// Count set bits (Brian Kernighan's algorithm)
function countBits(n) {
  let count = 0;
  while (n > 0) {
    n &= n - 1; // clears lowest set bit
    count++;
  }
  return count;
}

// XOR trick: find the single number (all others appear twice)
function singleNumber(nums) {
  return nums.reduce((acc, n) => acc ^ n, 0);
}

console.log(singleNumber([4, 1, 2, 1, 2])); // 4
```

```python
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0

def count_bits(n):
    count = 0
    while n:
        n &= n - 1
        count += 1
    return count

# Or simply: bin(n).count('1')

def single_number(nums):
    result = 0
    for n in nums:
        result ^= n
    return result

# Or: from functools import reduce; reduce(lambda a,b: a^b, nums)

print(single_number([4, 1, 2, 1, 2]))  # 4
```

```csharp
public bool IsPowerOfTwo(int n) => n > 0 && (n & (n - 1)) == 0;

public int CountBits(int n)
{
    int count = 0;
    while (n > 0) { n &= n - 1; count++; }
    return count;
}

public int SingleNumber(int[] nums)
{
    int result = 0;
    foreach (int n in nums) result ^= n;
    return result;
}
```

---

## Practice Problems

- [LeetCode 136 — Single Number](https://leetcode.com/problems/single-number/)
- [LeetCode 191 — Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/)
- [LeetCode 231 — Power of Two](https://leetcode.com/problems/power-of-two/)
- [LeetCode 268 — Missing Number](https://leetcode.com/problems/missing-number/)
- [LeetCode 338 — Counting Bits](https://leetcode.com/problems/counting-bits/)
