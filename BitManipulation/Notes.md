# Bit Manipulation — DSA Problem Solving Notes

## Core Intuition

Every integer is a sequence of bits. Bit operations are O(1) and faster than arithmetic. Problems that involve pairs, duplicates, subsets, or powers of two often have elegant bit solutions.

```
5  = 00000101
6  = 00000110
12 = 00001100
```

---

## Essential Operations

```csharp
// AND — both bits must be 1
a & b

// OR — at least one bit is 1
a | b

// XOR — bits differ (0 XOR 0 = 0, 1 XOR 1 = 0, 0 XOR 1 = 1)
a ^ b

// NOT — flip all bits
~a

// Left shift — multiply by 2^n
a << n   // a * 2^n

// Right shift — divide by 2^n (arithmetic, preserves sign)
a >> n   // a / 2^n
```

### Bit Tricks Cheat Sheet

```csharp
// Check if bit i is set
(n >> i) & 1 == 1

// Set bit i
n | (1 << i)

// Clear bit i
n & ~(1 << i)

// Toggle bit i
n ^ (1 << i)

// Check if n is power of 2
n > 0 && (n & (n - 1)) == 0

// Strip lowest set bit (e.g. 1100 → 1000)
n & (n - 1)

// Get lowest set bit only (e.g. 1100 → 0100)
n & (-n)

// Check if n is even/odd
(n & 1) == 0  // even
(n & 1) == 1  // odd

// Swap two numbers without temp
a ^= b; b ^= a; a ^= b;
```

**Why `n & (n-1)` strips the lowest set bit:**
```
n   = ...10100
n-1 = ...10011
AND = ...10000  ← lowest set bit gone
```

---

## Core Patterns

### 1. XOR — Find the Single Number

XOR is its own inverse: `a ^ a = 0`, `a ^ 0 = a`. XOR all elements — paired elements cancel, single element remains.

> LeetCode: [136. Single Number](https://leetcode.com/problems/single-number/), [137. Single Number II](https://leetcode.com/problems/single-number-ii/), [260. Single Number III](https://leetcode.com/problems/single-number-iii/)

```csharp
// Every element appears twice except one — find it
int SingleNumber(int[] nums)
{
    int result = 0;
    foreach (int n in nums) result ^= n;
    return result;
}
```

**Trace: [4, 1, 2, 1, 2]**
```
0 ^ 4 = 4
4 ^ 1 = 5
5 ^ 2 = 7
7 ^ 1 = 6  (1^1=0, 4^2=6? let's redo: 0^4=4, 4^1=5, 5^2=7, 7^1=6, 6^2=4) ✓
```

```csharp
// Every element appears three times except one — use bit counting
int SingleNumber(int[] nums)
{
    int result = 0;
    for (int i = 0; i < 32; i++)
    {
        int bitSum = 0;
        foreach (int n in nums) bitSum += (n >> i) & 1;
        result |= (bitSum % 3) << i; // leftover bit = the single number's bit
    }
    return result;
}
```

```csharp
// Two elements appear once, rest appear twice — find both
int[] SingleNumberIII(int[] nums)
{
    int xor = 0;
    foreach (int n in nums) xor ^= n; // xor = a ^ b

    // find any bit where a and b differ — use the lowest set bit
    int diff = xor & (-xor);

    int a = 0;
    foreach (int n in nums)
        if ((n & diff) != 0) a ^= n; // group by this bit — one group has a, other has b

    return [a, xor ^ a];
}
```

---

### 2. Count Set Bits (Popcount)

> LeetCode: [191. Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/), [338. Counting Bits](https://leetcode.com/problems/counting-bits/)

```csharp
// Count set bits in a single number
int HammingWeight(uint n)
{
    int count = 0;
    while (n != 0)
    {
        n &= n - 1; // strip lowest set bit each iteration
        count++;
    }
    return count;
}
```

```csharp
// Count bits for all numbers 0..n — use DP with the strip-lowest-bit relation
int[] CountBits(int n)
{
    var dp = new int[n + 1];
    for (int i = 1; i <= n; i++)
        dp[i] = dp[i & (i - 1)] + 1; // i with lowest bit stripped, plus 1
    return dp;
}
```

**Why `dp[i] = dp[i & (i-1)] + 1`:**
```
i = 6 = 110   →   i & (i-1) = 100 = 4
dp[6] = dp[4] + 1 = 1 + 1 = 2  ✓ (6 has two 1-bits)
```

---

### 3. Missing / Duplicate Number

> LeetCode: [268. Missing Number](https://leetcode.com/problems/missing-number/), [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

```csharp
// Missing number in [0..n] — XOR with indices cancels all present numbers
int MissingNumber(int[] nums)
{
    int result = nums.Length; // start with n
    for (int i = 0; i < nums.Length; i++)
        result ^= i ^ nums[i]; // XOR index and value — pairs cancel
    return result;
}
```

**Trace: [3, 0, 1] (n=3, missing=2)**
```
result = 3
i=0: 3 ^ 0 ^ 3 = 0
i=1: 0 ^ 1 ^ 0 = 1
i=2: 1 ^ 2 ^ 1 = 2  ✓
```

---

### 4. Power of Two / Power Checks

> LeetCode: [231. Power of Two](https://leetcode.com/problems/power-of-two/), [342. Power of Four](https://leetcode.com/problems/power-of-four/)

```csharp
bool IsPowerOfTwo(int n) => n > 0 && (n & (n - 1)) == 0;

// Power of four — must be power of 2 AND the set bit is at an even position
// Mask 0x55555555 = 01010101...01 — has 1s only at even bit positions
bool IsPowerOfFour(int n) => n > 0 && (n & (n - 1)) == 0 && (n & 0x55555555) != 0;
```

**Why `n & (n-1) == 0` for power of 2:**
```
8  = 1000
7  = 0111
AND = 0000  → power of 2 ✓

6  = 110
5  = 101
AND = 100  → not power of 2 ✓
```

---

### 5. Reverse Bits / Bit Manipulation on Integers

> LeetCode: [190. Reverse Bits](https://leetcode.com/problems/reverse-bits/)

```csharp
uint ReverseBits(uint n)
{
    uint result = 0;
    for (int i = 0; i < 32; i++)
    {
        result = (result << 1) | (n & 1); // shift result left, OR in the current bit
        n >>= 1;
    }
    return result;
}
```

**Trace: n = ...0101 (last 4 bits)**
```
i=0: result=0, n&1=1 → result=1,  n=...010
i=1: result=2, n&1=0 → result=2,  n=...01
i=2: result=4, n&1=1 → result=5,  n=...0
i=3: result=10,n&1=0 → result=10 = 1010  (reversed 0101) ✓
```

---

### 6. Subsets via Bitmask

Use an integer as a bitmask where bit `i` = "include element i". Enumerate all 2^n subsets.

> LeetCode: [78. Subsets](https://leetcode.com/problems/subsets/), [1986. Minimum Number of Work Sessions](https://leetcode.com/problems/minimum-number-of-work-sessions-to-finish-the-tasks/)

```csharp
IList<IList<int>> Subsets(int[] nums)
{
    int n = nums.Length;
    var res = new List<IList<int>>();

    for (int mask = 0; mask < (1 << n); mask++) // 2^n subsets
    {
        var subset = new List<int>();
        for (int i = 0; i < n; i++)
            if ((mask >> i & 1) == 1) // bit i is set → include nums[i]
                subset.Add(nums[i]);
        res.Add(subset);
    }
    return res;
}
```

**Trace: nums = [1, 2, 3]**
```
mask=000: {}
mask=001: {1}          (bit 0 set)
mask=010: {2}          (bit 1 set)
mask=011: {1,2}        (bits 0,1 set)
mask=100: {3}
mask=101: {1,3}
mask=110: {2,3}
mask=111: {1,2,3}
```

---

### 7. XOR in Range / Prefix XOR

XOR has the same prefix trick as sum: `XOR(l..r) = prefix[r] ^ prefix[l-1]`.

> LeetCode: [1310. XOR Queries of a Subarray](https://leetcode.com/problems/xor-queries-of-a-subarray/)

```csharp
int[] XorQueries(int[] arr, int[][] queries)
{
    int n = arr.Length;
    var prefix = new int[n + 1];
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] ^ arr[i];

    var res = new int[queries.Length];
    for (int i = 0; i < queries.Length; i++)
        res[i] = prefix[queries[i][1] + 1] ^ prefix[queries[i][0]];

    return res;
}
```

---

### 8. Bitmask DP

When state is a subset of n items (n ≤ 20), represent it as an integer bitmask. Classic use: Traveling Salesman, task assignment.

> LeetCode: [847. Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/), [1239. Maximum Length of a Concatenated String with Unique Characters](https://leetcode.com/problems/maximum-length-of-a-concatenated-string-with-unique-characters/)

```csharp
// Maximum length of concatenated string with unique characters
// Represent each string as a bitmask of which letters it uses
int MaxLength(IList<string> arr)
{
    var dp = new List<int> { 0 }; // dp[i] = bitmask of letters used in some valid concat

    int best = 0;
    foreach (var s in arr)
    {
        int mask = 0, len = 0;
        foreach (char c in s)
        {
            int bit = 1 << (c - 'a');
            if ((mask & bit) != 0) { mask = 0; break; } // duplicate in s itself
            mask |= bit;
            len++;
        }
        if (mask == 0) continue; // s had duplicates

        for (int i = dp.Count - 1; i >= 0; i--)
        {
            if ((dp[i] & mask) != 0) continue; // overlap with existing concat
            dp.Add(dp[i] | mask);
            best = Math.Max(best, int.PopCount(dp[i] | mask));
        }
    }
    return best;
}
```

---

## Bit Tricks Summary

```
Check bit i set:        (n >> i) & 1
Set bit i:              n | (1 << i)
Clear bit i:            n & ~(1 << i)
Toggle bit i:           n ^ (1 << i)
Strip lowest set bit:   n & (n - 1)      ← counts set bits, checks power of 2
Isolate lowest set bit: n & (-n)         ← split two XOR values
Is power of 2:          n > 0 && (n & (n-1)) == 0
Is even:                (n & 1) == 0
Count set bits:         use n & (n-1) loop, or int.PopCount(n) in C# (.NET 6+)
```

---

## Decision Guide

```
Find single element where rest appear twice?
  └─ XOR all elements — pairs cancel to 0

Find single element where rest appear three times?
  └─ Count bits mod 3 for each of 32 positions

Missing number in [0..n]?
  └─ XOR indices 0..n with array values

Check power of 2?
  └─ n > 0 && (n & (n-1)) == 0

Count set bits for all [0..n]?
  └─ DP: dp[i] = dp[i & (i-1)] + 1

Enumerate all subsets of n elements (n ≤ 20)?
  └─ Bitmask 0 to 2^n-1 — bit i = include element i

State is a subset of n items (n ≤ 20)?
  └─ Bitmask DP — dp[mask] = best result for this subset

XOR of subarray [l, r]?
  └─ Prefix XOR array — prefix[r+1] ^ prefix[l]

AVOID bit manipulation when:
  └─ n > 30 for bitmask enumeration — 2^30 is 1 billion, too slow
  └─ Problem is naturally array/string/tree — don't force bits
  └─ You'd need to explain a magic mask (0x55555555) — consider if there's a cleaner approach first
```

---

## Common Mistakes

1. **Signed vs unsigned shift** — `>>` in C# is arithmetic (sign-extending). For `uint`/`ulong` it's logical. If you're working with the sign bit, use `uint` or be careful with negative numbers.
2. **`1 << i` overflow** — `1` is a 32-bit int. For bit 31+, use `1L << i` (long) or `1u << i` (uint).
3. **Bitmask DP size** — `1 << n` states. For n=20 that's 1M; for n=25 it's 33M. Know your limits.
4. **Forgetting `n > 0` in power of 2 check** — `0 & (0-1) == 0` is true but 0 is not a power of 2.
5. **XOR swap pitfall** — `a ^= b; b ^= a; a ^= b` fails if `a` and `b` are the same variable/reference. Safe only for distinct variables.
6. **`int.PopCount` availability** — available in .NET 6+. For older versions use the `n & (n-1)` loop.

---

## Complexity Reference

| Operation | Time | Space |
|---|---|---|
| Single XOR pass | O(n) | O(1) |
| Count set bits (one number) | O(number of set bits) | O(1) |
| Count bits for all [0..n] | O(n) | O(n) |
| Enumerate all subsets | O(2^n × n) | O(2^n) |
| Bitmask DP | O(2^n × n) | O(2^n) |
| XOR prefix query | O(n) build, O(1) query | O(n) |