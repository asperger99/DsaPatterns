# Sliding Window — DSA Problem Solving Notes

## What is Sliding Window?

A technique where you maintain a window `[left, right]` over a sequence and slide it forward, avoiding recomputation by only updating what changed at the edges.

**Core intuition:** if you're computing something over a contiguous subarray/substring and re-scanning from scratch each time, a sliding window can cut it to O(n).

```
Array: [2, 1, 5, 1, 3, 2],  window size k=3

[2, 1, 5] 1, 3, 2   → sum = 8
 2,[1, 5, 1] 3, 2   → sum = 8 - 2 + 1 = 7   (slide: drop left, add right)
 2, 1,[5, 1, 3] 2   → sum = 7 - 1 + 3 = 9
 2, 1, 5,[1, 3, 2]  → sum = 9 - 5 + 2 = 6
```

---

## Two Types

### Fixed-size window
Window size `k` is given. Slide one step at a time, drop the leftmost, add the new right.

### Variable-size window
Window grows/shrinks based on a condition. Expand `right` to include more, shrink `left` when the window violates the constraint.

---

## Core Patterns

### 1. Fixed Window — Max/Min Sum of Size K

> LeetCode: [643. Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/), [1343. Number of Sub-arrays of Size K and Average Greater than Threshold](https://leetcode.com/problems/number-of-sub-arrays-of-size-k-and-average-greater-than-or-equal-to-threshold/)

```csharp
int MaxSumSubarray(int[] nums, int k)
{
    int sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i]; // build first window

    int best = sum;
    for (int i = k; i < nums.Length; i++)
    {
        sum += nums[i] - nums[i - k]; // slide: add right, drop left
        best = Math.Max(best, sum);
    }
    return best;
}
```

---

### 2. Variable Window — Longest Subarray/Substring Meeting a Condition

Expand `right` every iteration. Shrink `left` when the window is invalid.

> LeetCode: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/), [1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/), [424. Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)

```csharp
// Longest subarray with at most k zeros (Max Consecutive Ones III)
int LongestOnes(int[] nums, int k)
{
    int left = 0, zeros = 0, best = 0;

    for (int right = 0; right < nums.Length; right++)
    {
        if (nums[right] == 0) zeros++;

        while (zeros > k) // window invalid — shrink
        {
            if (nums[left] == 0) zeros--;
            left++;
        }

        best = Math.Max(best, right - left + 1);
    }
    return best;
}
```

```csharp
// Longest substring without repeating characters
int LengthOfLongestSubstring(string s)
{
    var last = new Dictionary<char, int>(); // char → last seen index
    int left = 0, best = 0;

    for (int right = 0; right < s.Length; right++)
    {
        if (last.TryGetValue(s[right], out int prev) && prev >= left)
            left = prev + 1; // jump left past the duplicate
        last[s[right]] = right;
        best = Math.Max(best, right - left + 1);
    }
    return best;
}
```

---

### 3. Variable Window — Shortest Subarray Meeting a Condition

Shrink `left` as much as possible while the window is still valid.

> LeetCode: [209. Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/), [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)

```csharp
// Minimum size subarray with sum >= target
int MinSubArrayLen(int target, int[] nums)
{
    int left = 0, sum = 0, best = int.MaxValue;

    for (int right = 0; right < nums.Length; right++)
    {
        sum += nums[right];

        while (sum >= target) // window valid — try to shrink
        {
            best = Math.Min(best, right - left + 1);
            sum -= nums[left++];
        }
    }
    return best == int.MaxValue ? 0 : best;
}
```

---

### 4. Window with HashMap — Track Contents

Use a map to track what's inside the window. Shrink when a constraint is violated.

> LeetCode: [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/), [438. Find All Anagrams](https://leetcode.com/problems/find-all-anagrams-in-a-string/), [567. Permutation in String](https://leetcode.com/problems/permutation-in-string/)

```csharp
// Minimum Window Substring
string MinWindow(string s, string t)
{
    var need = new Dictionary<char, int>();
    var have = new Dictionary<char, int>();
    foreach (char c in t) need[c] = need.GetValueOrDefault(c, 0) + 1;

    int left = 0, formed = 0, required = need.Count;
    int minLen = int.MaxValue, minLeft = 0;

    for (int right = 0; right < s.Length; right++)
    {
        char c = s[right];
        have[c] = have.GetValueOrDefault(c, 0) + 1;
        if (need.ContainsKey(c) && have[c] == need[c]) formed++;

        while (formed == required)
        {
            if (right - left + 1 < minLen) { minLen = right - left + 1; minLeft = left; }
            char l = s[left++];
            have[l]--;
            if (need.ContainsKey(l) && have[l] < need[l]) formed--;
        }
    }
    return minLen == int.MaxValue ? "" : s.Substring(minLeft, minLen);
}
```

```csharp
// Find all anagram start indices
IList<int> FindAnagrams(string s, string p)
{
    var need = new Dictionary<char, int>();
    var have = new Dictionary<char, int>();
    foreach (char c in p) need[c] = need.GetValueOrDefault(c, 0) + 1;

    var res = new List<int>();
    int left = 0, formed = 0, required = need.Count;

    for (int right = 0; right < s.Length; right++)
    {
        char c = s[right];
        have[c] = have.GetValueOrDefault(c, 0) + 1;
        if (need.ContainsKey(c) && have[c] == need[c]) formed++;

        if (right - left + 1 == p.Length) // fixed window size
        {
            if (formed == required) res.Add(left);
            char l = s[left++];
            if (need.ContainsKey(l) && have[l] == need[l]) formed--;
            have[l]--;
        }
    }
    return res;
}
```

---

### 5. Fixed Window with Monotonic Deque — Sliding Window Maximum

Track the maximum in the current window in O(1) using a deque that stores indices in decreasing order of value.

> LeetCode: [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

```csharp
int[] MaxSlidingWindow(int[] nums, int k)
{
    var dq = new LinkedList<int>(); // stores indices, front = max of window
    var res = new List<int>();

    for (int i = 0; i < nums.Length; i++)
    {
        // remove indices outside the window
        if (dq.Count > 0 && dq.First.Value < i - k + 1)
            dq.RemoveFirst();

        // remove smaller elements from back (they'll never be the max)
        while (dq.Count > 0 && nums[dq.Last.Value] < nums[i])
            dq.RemoveLast();

        dq.AddLast(i);

        if (i >= k - 1) res.Add(nums[dq.First.Value]);
    }
    return res.ToArray();
}
```

**Trace: nums = [1,3,-1,-3,5,3,6,7], k = 3**
```
i=0: dq=[0]
i=1: 3>1, pop 0 → dq=[1]
i=2: dq=[1,2],  window [1,3,-1]  → max = nums[1] = 3
i=3: dq=[1,2,3],window [3,-1,-3] → max = nums[1] = 3
i=4: 5>all, pop all → dq=[4],   window [-1,-3,5]  → max = 5
i=5: dq=[4,5],  window [-3,5,3]  → max = 5
i=6: 6>3, pop 5 → dq=[4,6],     window [5,3,6]   → max = 6
i=7: 7>all, pop all → dq=[7],   window [3,6,7]   → max = 7

Result: [3,3,5,5,6,7] ✓
```

---

### 6. Longest Repeating Character Replacement

Classic variable window where shrinking condition depends on a computed property of the window.

> LeetCode: [424. Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/)

```csharp
// Replace at most k chars to make all chars in window the same
int CharacterReplacement(string s, int k)
{
    var freq = new Dictionary<char, int>();
    int left = 0, maxFreq = 0, best = 0;

    for (int right = 0; right < s.Length; right++)
    {
        freq[s[right]] = freq.GetValueOrDefault(s[right], 0) + 1;
        maxFreq = Math.Max(maxFreq, freq[s[right]]);

        // window size - most frequent char count = chars to replace
        // if that > k, window is invalid
        int windowSize = right - left + 1;
        if (windowSize - maxFreq > k)
        {
            freq[s[left]]--;
            left++;
        }

        best = Math.Max(best, right - left + 1);
    }
    return best;
}
```

**Key insight:** we never need to shrink by more than 1 at a time. If the window was valid at size `n`, we only care about windows larger than `n` — so we just slide, not shrink aggressively.

---

## Template: Variable Window

Most variable window problems fit this shape:

```csharp
int left = 0;
// window state (sum, map, count, etc.)

for (int right = 0; right < n; right++)
{
    // 1. expand: add nums[right] to window state

    // 2. shrink: while window is invalid, remove nums[left] and left++
    while (/* window invalid */)
    {
        // remove nums[left] from state
        left++;
    }

    // 3. update answer (window is now valid)
    best = Math.Max(best, right - left + 1);
}
```

**Longest** problems → update answer after shrinking (inside the valid window).
**Shortest** problems → update answer inside the shrink loop (the moment window becomes valid).

---

### 7. Kadane's Algorithm — Maximum Subarray Sum

Kadane's is a special case of sliding window where the window expands or resets. At each position, you decide: extend the current subarray, or start fresh from here?

> LeetCode: [53. Maximum Subarray](https://leetcode.com/problems/maximum-subarray/), [152. Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/), [918. Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/)

```csharp
int MaxSubArray(int[] nums)
{
    int curr = nums[0], best = nums[0];

    for (int i = 1; i < nums.Length; i++)
    {
        curr = Math.Max(nums[i], curr + nums[i]); // extend or restart
        best = Math.Max(best, curr);
    }
    return best;
}
```

**The decision at each step:**
```
curr + nums[i]  →  extend the window (previous subarray was worth keeping)
nums[i]         →  restart from here (previous sum was dragging us down)

If curr < 0, adding it to nums[i] only hurts → restart.
If curr >= 0, it's always worth extending.
```

**Trace: [-2, 1, -3, 4, -1, 2, 1, -5, 4]**
```
i=0: curr=-2, best=-2
i=1: curr=max(1, -2+1)=1,   best=1
i=2: curr=max(-3, 1-3)=-2,  best=1
i=3: curr=max(4, -2+4)=4,   best=4
i=4: curr=max(-1, 4-1)=3,   best=4
i=5: curr=max(2, 3+2)=5,    best=5
i=6: curr=max(1, 5+1)=6,    best=6
i=7: curr=max(-5, 6-5)=1,   best=6
i=8: curr=max(4, 1+4)=5,    best=6

Result: 6  ✓  (subarray [4,-1,2,1])
```

### Variant: Maximum Product Subarray

Products are trickier — a negative × negative = positive, so you must track both max and min at each step.

```csharp
int MaxProduct(int[] nums)
{
    int max = nums[0], min = nums[0], best = nums[0];

    for (int i = 1; i < nums.Length; i++)
    {
        // a negative number flips max and min
        if (nums[i] < 0) (max, min) = (min, max);

        max = Math.Max(nums[i], max * nums[i]);
        min = Math.Min(nums[i], min * nums[i]);
        best = Math.Max(best, max);
    }
    return best;
}
```

**Trace: [2, 3, -2, 4]**
```
i=0: max=2,  min=2,  best=2
i=1: max=6,  min=3,  best=6
i=2: nums=-2 → swap → max=3,min=6
     max=max(-2, 3×-2)=-2, min=min(-2,6×-2)=-12, best=6
i=3: max=max(4,-2×4)=4, min=min(4,-12×4)=-48, best=6

Result: 6  ✓  (subarray [2,3])
```

---

## Decision Guide

```
Is window size fixed?
  └─ Yes → fixed window: sum += nums[right], sum -= nums[right - k]

Is window size variable?
  └─ All values non-negative?
      └─ Find LONGEST? → shrink with while loop, update best after
      └─ Find SHORTEST? → shrink with while loop, update best inside shrink loop
  └─ Contains negative numbers?
      └─ AVOID sliding window → use prefix sum + HashMap instead
         (shrinking left doesn't guarantee sum decreases, so the window loses its monotonic property)

Does the window need to track character/element counts?
  └─ Use HashMap inside the window

Need the MAX element of each window?
  └─ Monotonic deque (decreasing), O(1) max per window
```

---

## Common Mistakes

1. **Shrinking with `if` instead of `while`** — always use `while` to shrink; one step may not be enough to restore validity.
2. **Updating answer at the wrong place** — for longest, update after shrinking; for shortest, update inside the shrink loop.
3. **Off-by-one in window size** — window size is `right - left + 1`, not `right - left`.
4. **Not initializing the first window** — for fixed windows, build the first window in a separate loop before sliding.
5. **Jumping `left` past `right`** — add a guard `left <= right` if your shrink logic can overshoot.

---

## Complexity Reference

| Pattern | Time | Space |
|---|---|---|
| Fixed window (sum) | O(n) | O(1) |
| Variable window | O(n) | O(1) or O(k) |
| Window + HashMap | O(n) | O(k) |
| Sliding window maximum (deque) | O(n) | O(k) |