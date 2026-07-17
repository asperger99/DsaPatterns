# Prefix Sum — DSA Problem Solving Notes

## What is a Prefix Sum?

A prefix sum array where `prefix[i]` = sum of all elements from index `0` to `i-1`.
It lets you answer "what is the sum of elements from index `l` to `r`?" in **O(1)** after **O(n)** preprocessing — instead of summing every time which is O(n) per query.

```
arr:    [3,  1,  4,  1,  5,  9]
index:   0   1   2   3   4   5

prefix: [0,  3,  4,  8,  9, 14, 23]
index:   0   1   2   3   4   5   6

prefix[i] = arr[0] + arr[1] + ... + arr[i-1]
prefix[0] = 0  (empty prefix — always seed this)

Sum from l to r (inclusive) = prefix[r+1] - prefix[l]
Example: sum(1..3) = prefix[4] - prefix[1] = 9 - 3 = 6  ✓ (1+4+1=6)
```

---

## Building a Prefix Sum

```csharp
var prefix = new int[nums.Length + 1]; // prefix[0] = 0 by default
for (int i = 0; i < nums.Length; i++)
    prefix[i + 1] = prefix[i] + nums[i];

// range sum [l, r] inclusive
int sum = prefix[r + 1] - prefix[l];
```

**Always size `n+1` and leave `prefix[0] = 0`.** The formula `prefix[r+1] - prefix[l]` then works for all ranges including those starting at index 0.

---

## Core Patterns

### 1. Range Sum Query

Answer multiple "sum from l to r" queries in O(1) each after O(n) build.

> LeetCode: [303. Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/), [1480. Running Sum of 1d Array](https://leetcode.com/problems/running-sum-of-1d-array/)

```csharp
class NumArray
{
    int[] prefix;

    public NumArray(int[] nums)
    {
        prefix = new int[nums.Length + 1];
        for (int i = 0; i < nums.Length; i++)
            prefix[i + 1] = prefix[i] + nums[i];
    }

    public int SumRange(int l, int r) => prefix[r + 1] - prefix[l];
}
```

**Trace:**
```
nums:   [2, 7, 11, 3, 5]
prefix: [0, 2,  9, 20, 23, 28]

SumRange(1, 3) = prefix[4] - prefix[1] = 23 - 2 = 21  ✓ (7+11+3=21)
```

---

### 2. Subarray Sum Equals K

Count subarrays whose sum equals exactly `k`.
The trick: `sum[l..r] = k` → `prefix[r+1] - prefix[l] = k` → `prefix[l] = currentSum - k`.
So for each position, check if `(currentSum - k)` was seen before as a prefix sum.

> LeetCode: [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/)

```csharp
int SubarraySum(int[] nums, int k)
{
    var map = new Dictionary<int, int>();
    map[0] = 1; // empty prefix
    int sum = 0, count = 0;

    foreach (int n in nums)
    {
        sum += n;
        count += map.GetValueOrDefault(sum - k, 0);
        map[sum] = map.GetValueOrDefault(sum, 0) + 1;
    }
    return count;
}
```

**Trace with nums = [1, 2, 3], k = 3:**
```
map = {0:1}, sum = 0

n=1: sum=1, look for (1-3)=-2 → 0 hits. map={0:1, 1:1}
n=2: sum=3, look for (3-3)=0  → 1 hit (count=1). map={0:1,1:1,3:1}
n=3: sum=6, look for (6-3)=3  → 1 hit (count=2). map={0:1,1:1,3:1,6:1}

Result: 2  ✓ ([1,2] and [3])
```

**Why `map[0] = 1`?**
If the subarray starts at index 0, `prefix[l] = 0`. Without seeding 0, you'd miss it.

---

### 3. Subarray Sum Divisible by K

Count subarrays whose sum is divisible by `k`.
Key insight: `(prefix[r] - prefix[l]) % k == 0` → `prefix[r] % k == prefix[l] % k`.
Group prefixes by their remainder — two matching remainders mean the subarray between them is divisible by k.

> LeetCode: [974. Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/)

```csharp
int SubarraysDivByK(int[] nums, int k)
{
    var map = new Dictionary<int, int>();
    map[0] = 1;
    int sum = 0, count = 0;

    foreach (int n in nums)
    {
        sum += n;
        int rem = ((sum % k) + k) % k; // normalize negative remainders
        count += map.GetValueOrDefault(rem, 0);
        map[rem] = map.GetValueOrDefault(rem, 0) + 1;
    }
    return count;
}
```

**Why `((sum % k) + k) % k`?**
In C#, `-1 % 3 = -1` (not 2). Adding `k` before taking mod again normalizes it to positive.

---

### 4. Longest Subarray with Sum = K

When you want the **longest** subarray (not count), store the **first** index where each prefix sum appeared.

> LeetCode: [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/), [1208. Get Equal Substrings Within Budget](https://leetcode.com/problems/get-equal-substrings-within-budget/)

```csharp
int LongestSubarrayWithSum(int[] nums, int k)
{
    var map = new Dictionary<int, int>();
    map[0] = -1; // prefix sum 0 exists before index 0
    int sum = 0, best = 0;

    for (int i = 0; i < nums.Length; i++)
    {
        sum += nums[i];
        if (map.TryGetValue(sum - k, out int prev))
            best = Math.Max(best, i - prev);
        if (!map.ContainsKey(sum))
            map[sum] = i; // only store first occurrence
    }
    return best;
}
```

**First vs last index:**
- **First** → longest span (push left boundary as far left as possible)
- **Last** → most recent (sliding window)

---

### 5. Contiguous Array (Equal 0s and 1s)

Remap `0 → -1`. Now equal 0s and 1s in a subarray means sum = 0.
Reduces to: longest subarray with sum = 0 → prefix sum + first-seen index.

> LeetCode: [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/)

```csharp
int FindMaxLength(int[] nums)
{
    var map = new Dictionary<int, int>();
    map[0] = -1;
    int sum = 0, best = 0;

    for (int i = 0; i < nums.Length; i++)
    {
        sum += nums[i] == 0 ? -1 : 1;
        if (map.TryGetValue(sum, out int prev))
            best = Math.Max(best, i - prev);
        else
            map[sum] = i;
    }
    return best;
}
```

**Trace: [0, 1, 0, 0, 1, 1]**
```
i=0: sum=-1, map={0:-1, -1:0}
i=1: sum=0,  seen at -1 → len = 1-(-1) = 2
i=2: sum=-1, seen at 0  → len = 2-0 = 2
i=3: sum=-2, map={..., -2:3}
i=4: sum=-1, seen at 0  → len = 4-0 = 4
i=5: sum=0,  seen at -1 → len = 5-(-1) = 6  ← best

Result: 6 ✓
```

---

### 6. 2D Prefix Sum

Extend to matrices for O(1) rectangle sum queries after O(m×n) preprocessing.

> LeetCode: [304. Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/), [1314. Matrix Block Sum](https://leetcode.com/problems/matrix-block-sum/)

```
prefix[i][j] = sum of rectangle from (0,0) to (i-1, j-1)

Build:
prefix[i][j] = matrix[i-1][j-1] + prefix[i-1][j] + prefix[i][j-1] - prefix[i-1][j-1]

Query (r1,c1) to (r2,c2):
= prefix[r2+1][c2+1] - prefix[r1][c2+1] - prefix[r2+1][c1] + prefix[r1][c1]
```

```csharp
class NumMatrix
{
    int[,] prefix;

    public NumMatrix(int[][] matrix)
    {
        int m = matrix.Length, n = matrix[0].Length;
        prefix = new int[m + 1, n + 1];

        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                prefix[i, j] = matrix[i-1][j-1]
                              + prefix[i-1, j]
                              + prefix[i, j-1]
                              - prefix[i-1, j-1];
    }

    public int SumRegion(int r1, int c1, int r2, int c2)
        => prefix[r2+1, c2+1] - prefix[r1, c2+1] - prefix[r2+1, c1] + prefix[r1, c1];
}
```

**Visual (inclusion-exclusion):**
```
+-------+
|  A | B|       full - top strip - left strip + top-left (added back twice)
+----+--+
| C  | ?|
+-------+
```

---

### 7. Product Array (No Division)

Left pass builds prefix products, right pass multiplies in suffix products.

> LeetCode: [238. Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/)

```csharp
int[] ProductExceptSelf(int[] nums)
{
    int n = nums.Length;
    var res = new int[n];

    res[0] = 1;
    for (int i = 1; i < n; i++)
        res[i] = res[i - 1] * nums[i - 1]; // res[i] = product of everything left of i

    int right = 1;
    for (int i = n - 1; i >= 0; i--)
    {
        res[i] *= right;
        right *= nums[i];
    }
    return res;
}
```

**Trace: [1, 2, 3, 4]**
```
After left pass:  res = [1, 1, 2, 6]

right pass (right=1):
i=3: res[3] = 6×1=6,   right=4
i=2: res[2] = 2×4=8,   right=12
i=1: res[1] = 1×12=12, right=24
i=0: res[0] = 1×24=24

Result: [24, 12, 8, 6] ✓
```

---

## Decision Guide

```
Multiple range sum queries on static array?
  └─ 1D → prefix array, query in O(1)
  └─ 2D → 2D prefix, rectangle query in O(1)
  └─ Array gets updated between queries? → AVOID prefix sum, use Fenwick Tree / Segment Tree instead
     (prefix sum is static — any update forces a full O(n) rebuild)

Count subarrays with sum = k?
  └─ prefix + map, seed map[0] = 1
  └─ All values non-negative + no exact target needed? → sliding window is simpler

Find LONGEST subarray with sum = k?
  └─ prefix + first-seen index map, seed map[0] = -1
  └─ All values non-negative? → sliding window works and uses O(1) space

Sum divisible by k?
  └─ track remainders ((sum % k) + k) % k, match equal remainders

Equal 0s and 1s?
  └─ remap 0 → -1, find longest subarray with sum = 0

Product of all except self?
  └─ left prefix product × right suffix product (no division)
  └─ Contains zeros? → handle separately (zeros wipe out the product)
```

---

## Common Mistakes

1. **Seeding the map wrong** — count problems: `map[0] = 1`. Longest subarray: `map[0] = -1`. Mixing these up is the most common bug.
2. **Forgetting `prefix[0] = 0`** — size the array `n+1`. The formula breaks without it.
3. **Negative remainders** — C# `%` can return negative values. Always `((x % k) + k) % k`.
4. **Overwriting first index** — in longest subarray, only write `map[sum] = i` if the key isn't already there.
5. **Off-by-one in 2D** — prefix is `(m+1)×(n+1)`. Query subtracts `prefix[r1]` not `prefix[r1-1]`. Draw it out if confused.

---

## Complexity Reference

| Pattern | Time | Space |
|---|---|---|
| Build prefix array | O(n) | O(n) |
| Range sum query | O(1) | O(1) |
| Subarray sum = k (count) | O(n) | O(n) |
| Longest subarray sum = k | O(n) | O(n) |
| 2D prefix build | O(m×n) | O(m×n) |
| 2D range query | O(1) | O(1) |
| Product except self | O(n) | O(1) extra |