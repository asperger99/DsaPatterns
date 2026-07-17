# Binary Search — DSA Problem Solving Notes

## What is Binary Search?

Binary search eliminates half the search space at every step, reducing O(n) to O(log n).

**Core intuition:** you don't need a sorted array — you need a **monotonic condition**. If you can answer "is the answer at least X?" with true/false, and the answers look like `FFFFFFTTTTT`, binary search finds the first T in O(log n).

```
Sorted array: [1, 3, 5, 7, 9, 11, 13]
Find 7:

lo=0, hi=6 → mid=3 → arr[3]=7 → found ✓

Find 6:
lo=0, hi=6 → mid=3 → arr[3]=7 > 6 → hi=2
lo=0, hi=2 → mid=1 → arr[1]=3 < 6 → lo=2
lo=2, hi=2 → mid=2 → arr[2]=5 < 6 → lo=3
lo=3 > hi=2 → not found
```

---

## The Template

Binary search has subtle off-by-one bugs. Pick one template and stick to it.

```csharp
// Finds exact target. Returns -1 if not found.
int BinarySearch(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1;

    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2; // avoids overflow vs (lo+hi)/2

        if (nums[mid] == target) return mid;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

**Why `lo + (hi - lo) / 2`?**
`(lo + hi)` can overflow int when both are large. `lo + (hi - lo) / 2` is mathematically identical but safe.

---

## Core Patterns

### 1. Find Exact Target

> LeetCode: [704. Binary Search](https://leetcode.com/problems/binary-search/), [374. Guess Number Higher or Lower](https://leetcode.com/problems/guess-number-higher-or-lower/)

```csharp
int Search(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1;
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

---

### 2. Find Left / Right Boundary

When duplicates exist, find the first or last occurrence.

> LeetCode: [34. Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

```csharp
// First occurrence (left boundary)
int FirstOccurrence(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1, res = -1;
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) { res = mid; hi = mid - 1; } // keep going left
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return res;
}

// Last occurrence (right boundary)
int LastOccurrence(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1, res = -1;
    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) { res = mid; lo = mid + 1; } // keep going right
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return res;
}
```

---

### 3. Find Insert Position (Lower Bound)

Find the index where target would be inserted to keep the array sorted.
This is also "first index where `nums[i] >= target`".

> LeetCode: [35. Search Insert Position](https://leetcode.com/problems/search-insert-position/), [278. First Bad Version](https://leetcode.com/problems/first-bad-version/)

```csharp
int SearchInsert(int[] nums, int target)
{
    int lo = 0, hi = nums.Length; // hi = n (target might go at the end)
    while (lo < hi)               // note: lo < hi, not lo <= hi
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < target) lo = mid + 1;
        else hi = mid;             // nums[mid] >= target → mid could be the answer, keep it
    }
    return lo; // lo == hi, the insert position
}
```

**Trace: nums=[1,3,5,6], target=5**
```
lo=0, hi=4 → mid=2 → nums[2]=5 >= 5 → hi=2
lo=0, hi=2 → mid=1 → nums[1]=3 < 5  → lo=2
lo=2, hi=2 → exit

return 2 ✓
```

**`lo <= hi` vs `lo < hi`:**
```
lo <= hi  → use when returning mid directly (exact search)
lo < hi   → use when lo converges to the answer (boundary search)
            loop ends with lo == hi, return lo
```

---

### 4. Binary Search on Answer (Search Space)

The most powerful pattern. Instead of searching an array, you binary search on the **answer itself**.

**Identify it when:** "find the minimum/maximum value such that some condition holds."

> LeetCode: [875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/), [1011. Capacity to Ship Packages](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/), [410. Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/)

```csharp
// Koko Eating Bananas — minimum speed k to eat all piles in h hours
int MinEatingSpeed(int[] piles, int h)
{
    int lo = 1, hi = piles.Max();

    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (CanFinish(piles, mid, h)) hi = mid; // mid works, try smaller
        else lo = mid + 1;                       // mid too slow, go faster
    }
    return lo;
}

bool CanFinish(int[] piles, int speed, int h)
{
    int hours = 0;
    foreach (int p in piles) hours += (p + speed - 1) / speed; // ceil division
    return hours <= h;
}
```

```csharp
// Ship packages in D days — minimum capacity
int ShipWithinDays(int[] weights, int days)
{
    int lo = weights.Max(), hi = weights.Sum(); // min = heaviest item, max = ship all at once

    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (CanShip(weights, mid, days)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

bool CanShip(int[] weights, int cap, int days)
{
    int d = 1, curr = 0;
    foreach (int w in weights)
    {
        if (curr + w > cap) { d++; curr = 0; }
        curr += w;
    }
    return d <= days;
}
```

**The pattern for "binary search on answer":**
```
1. Define the search space [lo, hi]
   lo = smallest possible answer
   hi = largest possible answer

2. Write a CanDo(mid) function that checks if mid is feasible

3. Binary search:
   if CanDo(mid) → answer is mid or smaller → hi = mid
   else          → mid is too small         → lo = mid + 1

4. Return lo (the smallest feasible value)
```

---

### 5. Search in Rotated Sorted Array

Array is sorted but rotated. One half is always sorted — use that to decide which side to search.

> LeetCode: [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/), [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)

```csharp
int SearchRotated(int[] nums, int target)
{
    int lo = 0, hi = nums.Length - 1;

    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;

        if (nums[lo] <= nums[mid]) // left half is sorted
        {
            if (nums[lo] <= target && target < nums[mid]) hi = mid - 1; // target in left
            else lo = mid + 1;
        }
        else // right half is sorted
        {
            if (nums[mid] < target && target <= nums[hi]) lo = mid + 1; // target in right
            else hi = mid - 1;
        }
    }
    return -1;
}
```

```csharp
// Find minimum in rotated sorted array
int FindMin(int[] nums)
{
    int lo = 0, hi = nums.Length - 1;

    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) lo = mid + 1; // min is in right half
        else hi = mid;                            // mid could be the min
    }
    return nums[lo];
}
```

**Trace: [4, 5, 6, 7, 0, 1, 2], target = 0**
```
lo=0, hi=6, mid=3 → nums[3]=7
  left half [4,5,6,7] is sorted. target=0 not in [4,7] → lo=4

lo=4, hi=6, mid=5 → nums[5]=1
  left half [0,1] is sorted. target=0 in [0,1) → hi=4

lo=4, hi=4, mid=4 → nums[4]=0 == target → return 4 ✓
```

---

### 6. Search in 2D Matrix

> LeetCode: [74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/), [240. Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/)

```csharp
// Matrix where each row is sorted and first element of row > last of previous row
// → treat as flat sorted array
bool SearchMatrix(int[][] matrix, int target)
{
    int m = matrix.Length, n = matrix[0].Length;
    int lo = 0, hi = m * n - 1;

    while (lo <= hi)
    {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n]; // convert flat index to 2D
        if (val == target) return true;
        if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

```csharp
// Matrix II — each row and column sorted (not globally sorted)
// Start from top-right: go left if too big, go down if too small
bool SearchMatrixII(int[][] matrix, int target)
{
    int r = 0, c = matrix[0].Length - 1;

    while (r < matrix.Length && c >= 0)
    {
        if (matrix[r][c] == target) return true;
        if (matrix[r][c] > target) c--;
        else r++;
    }
    return false;
}
```

---

## Decision Guide

```
Exact value in sorted array?
  └─ Standard binary search, lo <= hi, return mid

First/last occurrence of target?
  └─ Left boundary: on match, save result and hi = mid - 1
  └─ Right boundary: on match, save result and lo = mid + 1

Insert position / first index >= target?
  └─ lo < hi loop, hi = mid on match, return lo

"Find min/max value such that condition holds"?
  └─ Binary search on answer space
  └─ Write CanDo(mid), search for smallest feasible mid

Array is rotated?
  └─ One half is always sorted — check which, then decide which side has target

2D matrix (globally sorted)?
  └─ Treat as flat array: matrix[mid/n][mid%n]

2D matrix (row + col sorted, not globally)?
  └─ Start top-right, eliminate row or column each step

AVOID binary search when:
  └─ Array is unsorted and sorting isn't allowed → linear scan
  └─ You need all occurrences → binary search finds one, then expand
  └─ Condition is not monotonic (no clear FFFTTTT pattern) → binary search won't work
```

---

## Common Mistakes

1. **`lo <= hi` vs `lo < hi`** — use `<=` for exact search (exit when not found). Use `<` for boundary search (exit when lo == hi = answer).
2. **`hi = mid` vs `hi = mid - 1`** — in boundary search, `hi = mid` keeps mid as a candidate. In exact search, `hi = mid - 1` discards mid since it wasn't the target.
3. **Overflow in mid** — always `lo + (hi - lo) / 2`, never `(lo + hi) / 2`.
4. **Wrong search space for "binary search on answer"** — `lo` must be the smallest possible valid answer (not 0), `hi` must be the largest.
5. **Infinite loop with `hi = mid`** — only safe when `lo < hi`. With `lo <= hi`, `hi = mid` can loop forever if `lo == mid`.
6. **Off-by-one in `hi` for insert position** — `hi = nums.Length` (not `Length - 1`) because target might insert at the end.

---

## Complexity Reference

| Pattern | Time | Space |
|---|---|---|
| Exact search | O(log n) | O(1) |
| Left / right boundary | O(log n) | O(1) |
| Binary search on answer | O(log(range) × O(check)) | O(1) |
| Rotated array search | O(log n) | O(1) |
| 2D matrix (globally sorted) | O(log(m×n)) | O(1) |
| 2D matrix (row+col sorted) | O(m + n) | O(1) |