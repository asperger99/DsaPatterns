# Two Pointers — DSA Problem Solving Notes

## What is Two Pointers?

Two pointers means maintaining two indices into an array or string and moving them based on some condition — avoiding nested loops and cutting O(n²) down to O(n).

**Core intuition:** when the brute force is "try every pair", ask yourself if there's a monotonic property that lets you move one pointer and discard a whole class of pairs.

---

## Three Variants

### Opposite ends (left + right)
Start one pointer at the front, one at the back. Move them toward each other.
Use when: sorted array, pair/triplet sum, palindrome check.

### Same direction (slow + fast)
Both start at the left, move right at different speeds.
Use when: remove duplicates, partition in-place, detect cycle in linked list.

### Two arrays / merge
One pointer per array, both moving forward.
Use when: merge sorted arrays, intersection, compare sequences.

---

## Core Patterns

### 1. Two Sum on Sorted Array

> LeetCode: [167. Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/), [15. 3Sum](https://leetcode.com/problems/3sum/), [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/)

```csharp
// Sorted array — find pair with sum = target
int[] TwoSum(int[] nums, int target)
{
    int l = 0, r = nums.Length - 1;

    while (l < r)
    {
        int sum = nums[l] + nums[r];
        if (sum == target) return [l + 1, r + 1]; // 1-indexed
        if (sum < target) l++;
        else r--;
    }
    return [];
}
```

**Why it works:** array is sorted. If `sum < target`, the only way to increase it is to move `l` right. If `sum > target`, move `r` left. Each step eliminates a row/column of the implicit n² matrix.

---

### 2. Three Sum

Fix one element, two-pointer the rest. Skip duplicates carefully.

> LeetCode: [15. 3Sum](https://leetcode.com/problems/3sum/), [18. 4Sum](https://leetcode.com/problems/4sum/)

```csharp
IList<IList<int>> ThreeSum(int[] nums)
{
    Array.Sort(nums);
    var res = new List<IList<int>>();

    for (int i = 0; i < nums.Length - 2; i++)
    {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate fixed element

        int l = i + 1, r = nums.Length - 1;
        while (l < r)
        {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0)
            {
                res.Add([nums[i], nums[l], nums[r]]);
                while (l < r && nums[l] == nums[l + 1]) l++; // skip duplicates
                while (l < r && nums[r] == nums[r - 1]) r--; // skip duplicates
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

### 3. Container With Most Water / Trapping Rain Water

> LeetCode: [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/), [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/)

```csharp
// Container With Most Water
int MaxArea(int[] height)
{
    int l = 0, r = height.Length - 1, best = 0;

    while (l < r)
    {
        int area = Math.Min(height[l], height[r]) * (r - l);
        best = Math.Max(best, area);
        if (height[l] < height[r]) l++; // move the shorter side — only way to possibly improve
        else r--;
    }
    return best;
}
```

```csharp
// Trapping Rain Water
int Trap(int[] height)
{
    int l = 0, r = height.Length - 1;
    int maxL = 0, maxR = 0, water = 0;

    while (l < r)
    {
        if (height[l] < height[r])
        {
            if (height[l] >= maxL) maxL = height[l];
            else water += maxL - height[l];
            l++;
        }
        else
        {
            if (height[r] >= maxR) maxR = height[r];
            else water += maxR - height[r];
            r--;
        }
    }
    return water;
}
```

---

### 4. Palindrome Check / Two Pointers on String

> LeetCode: [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/), [680. Valid Palindrome II](https://leetcode.com/problems/valid-palindrome-ii/)

```csharp
bool IsPalindrome(string s)
{
    int l = 0, r = s.Length - 1;
    while (l < r)
    {
        while (l < r && !char.IsLetterOrDigit(s[l])) l++;
        while (l < r && !char.IsLetterOrDigit(s[r])) r--;
        if (char.ToLower(s[l]) != char.ToLower(s[r])) return false;
        l++; r--;
    }
    return true;
}
```

```csharp
// Valid Palindrome II — allow at most one deletion
bool ValidPalindrome(string s)
{
    int l = 0, r = s.Length - 1;
    while (l < r)
    {
        if (s[l] != s[r])
            return IsPalin(s, l + 1, r) || IsPalin(s, l, r - 1);
        l++; r--;
    }
    return true;
}

bool IsPalin(string s, int l, int r)
{
    while (l < r) { if (s[l++] != s[r--]) return false; }
    return true;
}
```

---

### 5. Remove Duplicates / In-Place Partition (Slow + Fast)

One pointer (`w`) tracks where to write next. The other (`r`) scans ahead.

> LeetCode: [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/), [27. Remove Element](https://leetcode.com/problems/remove-element/), [75. Sort Colors](https://leetcode.com/problems/sort-colors/)

```csharp
// Remove duplicates from sorted array in-place
int RemoveDuplicates(int[] nums)
{
    int w = 1; // write pointer
    for (int r = 1; r < nums.Length; r++)
        if (nums[r] != nums[r - 1])
            nums[w++] = nums[r];
    return w;
}
```

```csharp
// Remove all occurrences of val in-place
int RemoveElement(int[] nums, int val)
{
    int w = 0;
    foreach (int n in nums)
        if (n != val) nums[w++] = n;
    return w;
}
```

```csharp
// Sort Colors (Dutch National Flag) — 3-way partition
void SortColors(int[] nums)
{
    int l = 0, mid = 0, r = nums.Length - 1;

    while (mid <= r)
    {
        if (nums[mid] == 0) swap(nums, l++, mid++);
        else if (nums[mid] == 1) mid++;
        else swap(nums, mid, r--); // don't increment mid — swapped element is unknown
    }
}

void swap(int[] nums, int i, int j) => (nums[i], nums[j]) = (nums[j], nums[i]);
```

**Trace (Sort Colors): [2,0,2,1,1,0]**
```
l=0, mid=0, r=5

mid=0: nums[0]=2 → swap(mid,r) → [0,0,2,1,1,2], r=4
mid=0: nums[0]=0 → swap(l,mid) → [0,0,2,1,1,2], l=1,mid=1
mid=1: nums[1]=0 → swap(l,mid) → [0,0,2,1,1,2], l=2,mid=2
mid=2: nums[2]=2 → swap(mid,r) → [0,0,1,1,2,2], r=3
mid=2: nums[2]=1 → mid=3
mid=3: nums[3]=1 → mid=4
mid=4 > r=3, done

Result: [0,0,1,1,2,2] ✓
```

---

### 6. Merge Two Sorted Arrays

One pointer per array, always pick the smaller.

> LeetCode: [88. Merge Sorted Array](https://leetcode.com/problems/merge-sorted-array/), [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)

```csharp
// Merge nums2 into nums1 in-place (fill from the back to avoid overwriting)
void Merge(int[] nums1, int m, int[] nums2, int n)
{
    int i = m - 1, j = n - 1, k = m + n - 1;

    while (j >= 0)
    {
        if (i >= 0 && nums1[i] > nums2[j])
            nums1[k--] = nums1[i--];
        else
            nums1[k--] = nums2[j--];
    }
}
```

---

### 7. Fast and Slow Pointers (Floyd's Cycle Detection)

> LeetCode: [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/), [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/), [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/)

```csharp
// Detect cycle in linked list
bool HasCycle(ListNode head)
{
    var slow = head;
    var fast = head;

    while (fast != null && fast.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

```csharp
// Find cycle entry point
ListNode DetectCycle(ListNode head)
{
    var slow = head;
    var fast = head;

    while (fast != null && fast.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast)
        {
            slow = head; // reset one pointer to head
            while (slow != fast) { slow = slow.next; fast = fast.next; }
            return slow; // entry point
        }
    }
    return null;
}
```

**Example:**
```
Nodes:  0 → 1 → 2 → 3 → 4
                 ↑        |
                 └────────┘   cycle entry = node 2, cycle = 2→3→4→2
```

**Phase 1 — detect the cycle (find meeting point):**
```
        slow    fast
start:   0       0
step 1:  1       2      (fast jumps 2)
step 2:  2       4      (fast jumps 2)
step 3:  3       3      (fast: 4→2→3... wait, 4.next=2, 2.next=3)  ← MEET at node 3
```

They meet inside the cycle at node 3. But node 3 is NOT the entry — the entry is node 2.

**Phase 2 — find the entry:**

Reset `slow` to head (node 0). Keep `fast` at node 3. Now move BOTH one step at a time:
```
        slow    fast
start:   0       3
step 1:  1       4
step 2:  2       2      ← MEET at node 2 = cycle entry ✓
```

**Why does this work?**
```
d = steps from head to entry    = 2  (0→1→2)
c = cycle length                = 3  (2→3→4→2)

When fast and slow meet, slow has taken S steps.
fast took 2S steps but also lapped the cycle, so:
  2S = S + c  →  S = c  (slow walked exactly one cycle)

The meeting point is (S - d) steps into the cycle from the entry.
Steps remaining from meeting point back to entry = c - (S - d) = d.

So from the meeting point, it takes exactly d more steps to reach the entry.
And from the head, it also takes exactly d steps to reach the entry.

→ reset one pointer to head, walk both at speed 1 → they meet at the entry after d steps.
```

---

## Template

### Opposite ends
```csharp
int l = 0, r = nums.Length - 1;
while (l < r)
{
    if (/* condition met */) { /* record answer */ l++; r--; }
    else if (/* need bigger */) l++;
    else r--;
}
```

### Slow + Fast (in-place write)
```csharp
int w = 0; // write pointer
for (int r = 0; r < nums.Length; r++)
{
    if (/* keep nums[r] */)
        nums[w++] = nums[r];
}
```

---

## Decision Guide

```
Sorted array, find pair/triplet with sum?
  └─ Opposite ends two pointers
  └─ Array NOT sorted? → AVOID two pointers, use HashMap for O(n) instead
     (sorting costs O(n log n) and loses original indices)

Check or build palindrome?
  └─ Opposite ends, skip non-alphanumeric

Remove/filter elements in-place?
  └─ Slow (write) + fast (read) pointers
  └─ Need to preserve order of removed elements? → use a separate list, not in-place

Partition array into groups in-place?
  └─ Dutch National Flag (3 pointers: l, mid, r)
  └─ More than 3 groups? → AVOID, use counting sort instead

Merge two sorted sequences?
  └─ One pointer per sequence, compare and advance
  └─ Sequences not sorted? → sort first or use a heap for k-way merge

Cycle in linked list?
  └─ Fast + slow (Floyd's)
  └─ Need cycle length? → once slow==fast, keep one fixed and count steps until they meet again

Find duplicate / missing number?
  └─ Fast + slow (treat array as linked list via index jumps)
  └─ Multiple duplicates? → AVOID fast+slow, use HashSet or XOR trick instead
```

---

## Common Mistakes

1. **Using two pointers on unsorted data for sum problems** — opposite-ends only works because sorted order gives a monotonic property. Sort first.
2. **Forgetting to skip duplicates in 3Sum** — after finding a valid triplet, advance both pointers past all equal values before `l++; r--`.
3. **`while` vs `if` for shrinking** — when you want the smallest valid window, use `while`; `if` stops too early.
4. **Fast pointer null check** — always check `fast != null && fast.next != null` before `fast.next.next`.
5. **Not incrementing `mid` after swapping with `r`** in Dutch National Flag — the swapped element is unknown so you must re-examine it.

---

## Complexity Reference

| Pattern | Time | Space |
|---|---|---|
| Two sum (sorted) | O(n) | O(1) |
| Three sum | O(n²) | O(1) |
| Container / rain water | O(n) | O(1) |
| Remove duplicates | O(n) | O(1) |
| Dutch National Flag | O(n) | O(1) |
| Merge sorted arrays | O(m+n) | O(1) |
| Fast + slow (cycle) | O(n) | O(1) |