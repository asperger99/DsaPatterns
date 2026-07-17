# Sorting — DSA Problem Solving Notes

## What is Sorting?

Sorting arranges elements in order. In interviews you rarely implement a sort from scratch — but you need to know:
1. Which sort fits which situation
2. How to use C#'s built-in sort with custom comparators
3. When sorting enables a simpler algorithm (two pointers, binary search, greedy)

**Core intuition:** if a problem feels O(n²) with a brute force scan, ask "what if the array were sorted?" Sorting costs O(n log n) but often unlocks O(n) or O(log n) solutions.

---

## C# Built-in Sort

```csharp
// Array sort — in-place, O(n log n), unstable (introsort)
Array.Sort(arr);
Array.Sort(arr, (a, b) => a.CompareTo(b));   // ascending
Array.Sort(arr, (a, b) => b.CompareTo(a));   // descending

// Sort by custom key
Array.Sort(intervals, (a, b) => a[0] != b[0] ? a[0] - b[0] : a[1] - b[1]); // by start, then end

// List sort
list.Sort();
list.Sort((a, b) => a.CompareTo(b));

// LINQ — returns new sorted sequence, does NOT sort in-place
var sorted = arr.OrderBy(x => x).ToArray();
var sorted = arr.OrderByDescending(x => x).ToArray();
var sorted = arr.OrderBy(x => x.Length).ThenBy(x => x).ToArray(); // multi-key

// Sort strings by length then alphabetically
words.Sort((a, b) => a.Length != b.Length ? a.Length - b.Length : string.Compare(a, b));
```

**Comparator contract:** return negative if `a` comes first, positive if `b` comes first, zero if equal.

---

## Sorting Algorithms

### Merge Sort — O(n log n), stable, O(n) space

Divide array in half, sort each half, merge. Guaranteed O(n log n) in all cases.
Best for: linked lists, external sorting, when stability matters.

```csharp
void MergeSort(int[] arr, int l, int r)
{
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    MergeSort(arr, l, mid);
    MergeSort(arr, mid + 1, r);
    Merge(arr, l, mid, r);
}

void Merge(int[] arr, int l, int mid, int r)
{
    var left  = arr[l..(mid + 1)];
    var right = arr[(mid + 1)..(r + 1)];
    int i = 0, j = 0, k = l;

    while (i < left.Length && j < right.Length)
        arr[k++] = left[i] <= right[j] ? left[i++] : right[j++];

    while (i < left.Length)  arr[k++] = left[i++];
    while (j < right.Length) arr[k++] = right[j++];
}
```

---

### Quick Sort — O(n log n) avg, O(n²) worst, in-place, unstable

Partition around a pivot. Everything < pivot goes left, > pivot goes right. Recurse.
Best for: in-place sorting, average-case performance.

```csharp
void QuickSort(int[] arr, int l, int r)
{
    if (l >= r) return;
    int pivot = Partition(arr, l, r);
    QuickSort(arr, l, pivot - 1);
    QuickSort(arr, pivot + 1, r);
}

int Partition(int[] arr, int l, int r)
{
    int pivot = arr[r]; // pick last element as pivot
    int i = l - 1;

    for (int j = l; j < r; j++)
        if (arr[j] <= pivot)
            (arr[++i], arr[j]) = (arr[j], arr[i]);

    (arr[i + 1], arr[r]) = (arr[r], arr[i + 1]);
    return i + 1;
}
```

**Trace: [3,6,8,10,1,2,1], pivot=1 (last)**
```
i=-1, pivot=1

j=0: arr[0]=3 > 1 → skip
j=1: arr[1]=6 > 1 → skip
j=2: arr[2]=8 > 1 → skip
j=3: arr[3]=10 > 1 → skip
j=4: arr[4]=1 <= 1 → i=0, swap(0,4) → [1,6,8,10,3,2,1]
j=5: arr[5]=2 > 1 → skip

swap(i+1=1, r=6) → [1,1,8,10,3,2,6]
pivot at index 1
```

---

### Counting Sort — O(n + k), stable, O(k) space

Count occurrences of each value, reconstruct. Only works for integer keys in a known range `[0, k]`.

```csharp
int[] CountingSort(int[] arr, int maxVal)
{
    var count = new int[maxVal + 1];
    foreach (int n in arr) count[n]++;

    var res = new int[arr.Length];
    int idx = 0;
    for (int i = 0; i <= maxVal; i++)
        while (count[i]-- > 0) res[idx++] = i;

    return res;
}
```

---

### Bucket Sort — O(n + k) avg, O(n) space

Distribute elements into buckets, sort each bucket, concatenate.
Best for: uniformly distributed floats, when values fit in a bounded range.

```csharp
double[] BucketSort(double[] arr)
{
    int n = arr.Length;
    var buckets = new List<double>[n];
    for (int i = 0; i < n; i++) buckets[i] = new List<double>();

    foreach (double x in arr)
        buckets[(int)(x * n)].Add(x); // map [0,1) → bucket index

    foreach (var b in buckets) b.Sort();

    int idx = 0;
    foreach (var b in buckets)
        foreach (double x in b) arr[idx++] = x;

    return arr;
}
```

---

### Radix Sort — O(d × n), stable, O(n + k) space

Sort by digits from least significant to most significant. `d` = number of digits.
Best for: large integers where comparison-based sorts are slow.

```csharp
void RadixSort(int[] arr)
{
    int max = arr.Max();
    for (int exp = 1; max / exp > 0; exp *= 10)
        CountByDigit(arr, exp);
}

void CountByDigit(int[] arr, int exp)
{
    int n = arr.Length;
    var output = new int[n];
    var count = new int[10];

    foreach (int x in arr) count[(x / exp) % 10]++;
    for (int i = 1; i < 10; i++) count[i] += count[i - 1]; // prefix sum for stable placement

    for (int i = n - 1; i >= 0; i--) // traverse right to left for stability
    {
        int digit = (arr[i] / exp) % 10;
        output[--count[digit]] = arr[i];
    }
    Array.Copy(output, arr, n);
}
```

---

## Sorting-Enabled Patterns

### Interval Merging

Sort by start time. Greedily merge overlapping intervals.

> LeetCode: [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/), [57. Insert Interval](https://leetcode.com/problems/insert-interval/)

```csharp
int[][] Merge(int[][] intervals)
{
    Array.Sort(intervals, (a, b) => a[0] - b[0]);
    var res = new List<int[]>();

    foreach (var interval in intervals)
    {
        if (res.Count == 0 || res[^1][1] < interval[0])
            res.Add(interval); // no overlap — add new
        else
            res[^1][1] = Math.Max(res[^1][1], interval[1]); // overlap — extend end
    }
    return res.ToArray();
}
```

---

### Meeting Rooms / Non-overlapping Intervals

> LeetCode: [252. Meeting Rooms](https://leetcode.com/problems/meeting-rooms/), [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/)

```csharp
// Minimum intervals to remove so none overlap
// Greedy: sort by end time, always keep interval that ends earliest
int EraseOverlapIntervals(int[][] intervals)
{
    Array.Sort(intervals, (a, b) => a[1] - b[1]); // sort by end time
    int removed = 0, lastEnd = int.MinValue;

    foreach (var iv in intervals)
    {
        if (iv[0] >= lastEnd) lastEnd = iv[1]; // no overlap — keep it
        else removed++;                          // overlap — remove current (it ends later)
    }
    return removed;
}
```

---

### Sort + Two Pointers

Sorting unlocks two-pointer patterns that only work on ordered data.

> LeetCode: [15. 3Sum](https://leetcode.com/problems/3sum/), [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/)

```csharp
// 3Sum — sort first, then fix one element and two-pointer the rest
IList<IList<int>> ThreeSum(int[] nums)
{
    Array.Sort(nums);
    var res = new List<IList<int>>();

    for (int i = 0; i < nums.Length - 2; i++)
    {
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        int l = i + 1, r = nums.Length - 1;
        while (l < r)
        {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0)
            {
                res.Add([nums[i], nums[l], nums[r]]);
                while (l < r && nums[l] == nums[l + 1]) l++;
                while (l < r && nums[r] == nums[r - 1]) r--;
                l++; r--;
            }
            else if (sum < 0) l++;
            else r--;
        }
    }
    return res;
}
```

---

### Custom Comparator Problems

When the sort order itself is the trick.

> LeetCode: [179. Largest Number](https://leetcode.com/problems/largest-number/), [280. Wiggle Sort](https://leetcode.com/problems/wiggle-sort/)

```csharp
// Largest Number — arrange nums to form the largest possible number
string LargestNumber(int[] nums)
{
    var strs = nums.Select(n => n.ToString()).ToArray();
    Array.Sort(strs, (a, b) => string.Compare(b + a, a + b)); // compare concatenations

    if (strs[0] == "0") return "0"; // all zeros edge case
    return string.Concat(strs);
}
```

**Why compare `b+a` vs `a+b`?**
```
"9" vs "34": "934" > "349" → 9 should come first → sort descending by concatenation
"3" vs "30": "330" > "303" → 3 should come first
```

---

### Counting Sort for Frequency Problems

> LeetCode: [451. Sort Characters By Frequency](https://leetcode.com/problems/sort-characters-by-frequency/)

```csharp
string FrequencySort(string s)
{
    var freq = new Dictionary<char, int>();
    foreach (char c in s) freq[c] = freq.GetValueOrDefault(c, 0) + 1;

    // bucket by frequency — index = frequency
    var buckets = new List<char>[s.Length + 1];
    foreach (var (c, f) in freq)
    {
        buckets[f] ??= new List<char>();
        buckets[f].Add(c);
    }

    var sb = new System.Text.StringBuilder();
    for (int f = buckets.Length - 1; f >= 1; f--)
        if (buckets[f] != null)
            foreach (char c in buckets[f])
                sb.Append(new string(c, f));

    return sb.ToString();
}
```

---

## Algorithm Comparison

| Algorithm | Best | Average | Worst | Space | Stable | Use When |
|---|---|---|---|---|---|---|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Stability needed, linked list |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No | In-place, average perf |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Guaranteed O(n log n), in-place |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes | Small integer range |
| Radix Sort | O(d×n) | O(d×n) | O(d×n) | O(n+k) | Yes | Large integers, fixed digits |
| Bucket Sort | O(n+k) | O(n+k) | O(n²) | O(n) | Yes | Uniform distribution |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Nearly sorted, tiny n |

**C# `Array.Sort` uses introsort** — hybrid of quicksort, heapsort, and insertion sort. O(n log n) guaranteed.

---

## Decision Guide

```
General purpose sort?
  └─ Array.Sort() — just use it

Need stable sort?
  └─ LINQ OrderBy (stable) or Merge Sort
  └─ Array.Sort is NOT stable

Need to sort by multiple keys?
  └─ LINQ: .OrderBy(x => x.key1).ThenBy(x => x.key2)
  └─ Comparator: return key1 diff first, fallback to key2 diff

Elements are integers in range [0, k]?
  └─ Counting Sort — O(n + k), beats comparison sorts

Need largest/smallest K elements, not full sort?
  └─ Heap — O(n log k), faster than full sort O(n log n)

Overlapping intervals?
  └─ Sort by start time → greedy merge

Non-overlapping intervals (keep max)?
  └─ Sort by end time → greedy keep earliest ending

Custom ordering (largest number, etc.)?
  └─ Comparator comparing concatenations or derived values

AVOID sorting when:
  └─ Array is already sorted → just binary search / two pointers
  └─ You only need Kth element → quickselect O(n) avg
  └─ Input is a stream → can't sort, use heap
  └─ n is huge and k is tiny → partial sort / heap beats full sort
```

---

## Common Mistakes

1. **Unstable sort losing relative order** — `Array.Sort` is unstable. If relative order of equal elements matters, use `OrderBy` (LINQ, stable) or add a tiebreaker to the comparator.
2. **Integer overflow in comparator** — `a - b` can overflow for large values. Use `a.CompareTo(b)` instead of `a - b`.
3. **Sorting when you need Kth element** — use quickselect or a heap for O(n) or O(n log k) instead of full O(n log n) sort.
4. **Wrong interval sort key** — merge intervals needs sort by **start**, non-overlapping needs sort by **end**. Mixing them gives wrong greedy decisions.
5. **Mutating original array** — `Array.Sort` sorts in-place. If you need the original, copy first: `var copy = (int[])arr.Clone()`.
6. **Comparator not returning 0 for equals** — if your comparator never returns 0, equal elements may be sorted inconsistently across platforms.

---

## Complexity Reference

| Scenario | Algorithm | Time |
|---|---|---|
| General sort | Array.Sort (introsort) | O(n log n) |
| Kth largest/smallest | Heap or Quickselect | O(n log k) / O(n) |
| Integer range [0, k] | Counting Sort | O(n + k) |
| Sort + merge intervals | Sort + linear scan | O(n log n) |
| Sort + two pointers | Sort + O(n) | O(n log n) |