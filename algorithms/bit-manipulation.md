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

---

## Practice Problems

- [LeetCode 136 — Single Number](https://leetcode.com/problems/single-number/)
- [LeetCode 191 — Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/)
- [LeetCode 231 — Power of Two](https://leetcode.com/problems/power-of-two/)
- [LeetCode 268 — Missing Number](https://leetcode.com/problems/missing-number/)
- [LeetCode 338 — Counting Bits](https://leetcode.com/problems/counting-bits/)
