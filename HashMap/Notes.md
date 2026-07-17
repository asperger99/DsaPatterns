# HashMap — DSA Problem Solving Notes

## What is a HashMap?

A HashMap stores **key → value** pairs with O(1) average-case lookup, insert, and delete.
Use it whenever you need to remember something you've seen before and look it up fast.

**Core intuition:** if you're ever scanning an array twice to find a relationship between elements, a HashMap can usually cut it to one pass.

---

## C# Essentials

```csharp
var map = new Dictionary<int, int>();

map[key] = value;
map.ContainsKey(key);
map.TryGetValue(key, out int val);
map.GetValueOrDefault(key, 0);
map.Remove(key);

// increment count (most common pattern)
map[key] = map.GetValueOrDefault(key, 0) + 1;

// HashSet — existence only, no count
var seen = new HashSet<int>();
seen.Add(x);
seen.Contains(x);
```

---

## Core Patterns

### 1. Frequency Count

Count occurrences of elements. Foundation for many problems.

> LeetCode: [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/), [242. Valid Anagram](https://leetcode.com/problems/valid-anagram/), [49. Group Anagrams](https://leetcode.com/problems/group-anagrams/)

```csharp
var freq = new Dictionary<char, int>();
foreach (char c in s)
    freq[c] = freq.GetValueOrDefault(c, 0) + 1;
```

```csharp
// Valid Anagram
bool IsAnagram(string s, string t)
{
    if (s.Length != t.Length) return false;

    var map = new Dictionary<char, int>();
    foreach (char c in s) map[c] = map.GetValueOrDefault(c, 0) + 1;
    foreach (char c in t)
    {
        if (!map.ContainsKey(c) || map[c] == 0) return false;
        map[c]--;
    }
    return true;
}
```

---

### 2. Two Sum / Complement Lookup

Store what you've seen so far. For each element, check if its complement already exists.

> LeetCode: [1. Two Sum](https://leetcode.com/problems/two-sum/), [167. Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/)

```csharp
int[] TwoSum(int[] nums, int target)
{
    var map = new Dictionary<int, int>(); // val → index

    for (int i = 0; i < nums.Length; i++)
    {
        int need = target - nums[i];
        if (map.ContainsKey(need)) return [map[need], i];
        map[nums[i]] = i;
    }
    return [];
}
```

**Pattern generalizes to:**
- Three Sum → fix one element, two-sum the rest
- Subarray sum equals K → prefix sum + HashMap

---

### 3. Prefix Sum + HashMap

Track running sums. Use HashMap to answer "have I seen this sum before?" in O(1).

> LeetCode: [560. Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/), [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/)

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

**Why it works:**
```
sum[0..j] - sum[0..i] = k
→ sum[0..i] = currentSum - k
→ if that prefix exists in the map, a valid subarray ends at j
```

---

### 4. Sliding Window + HashMap

Use a HashMap to track the window's contents and shrink/expand as needed.

> LeetCode: [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/), [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/), [438. Find All Anagrams](https://leetcode.com/problems/find-all-anagrams-in-a-string/)

```csharp
// Longest substring without repeating characters
int LengthOfLongestSubstring(string s)
{
    var last = new Dictionary<char, int>(); // char → last seen index
    int best = 0, left = 0;

    for (int i = 0; i < s.Length; i++)
    {
        if (last.TryGetValue(s[i], out int prev) && prev >= left)
            left = prev + 1;
        last[s[i]] = i;
        best = Math.Max(best, i - left + 1);
    }
    return best;
}
```

```csharp
// Minimum window substring
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

---

### 5. Grouping / Bucketing

Use a computed key to group elements that share a property.

> LeetCode: [49. Group Anagrams](https://leetcode.com/problems/group-anagrams/), [128. Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/)

```csharp
// Group Anagrams
IList<IList<string>> GroupAnagrams(string[] strs)
{
    var map = new Dictionary<string, List<string>>();
    foreach (string s in strs)
    {
        var key = string.Concat(s.OrderBy(c => c));
        if (!map.ContainsKey(key)) map[key] = new();
        map[key].Add(s);
    }
    return new List<IList<string>>(map.Values);
}
```

```csharp
// Longest Consecutive Sequence — O(n)
// Only start counting from the beginning of a sequence (n-1 not in set)
int LongestConsecutive(int[] nums)
{
    var set = new HashSet<int>(nums);
    int best = 0;

    foreach (int n in set)
    {
        if (set.Contains(n - 1)) continue;
        int len = 1;
        while (set.Contains(n + len)) len++;
        best = Math.Max(best, len);
    }
    return best;
}
```

---

### 6. Index / Position Tracking

Store the first or last index where something occurred to calculate spans or detect patterns.

> LeetCode: [525. Contiguous Array](https://leetcode.com/problems/contiguous-array/)

```csharp
// Longest subarray with equal 0s and 1s (treat 0 as -1, find longest subarray with sum = 0)
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

**Rule:** store **first** index → longest span. Store **last** index → most recent (sliding window).

---

### 7. Two-Pass HashMap

First pass: build the map. Second pass: query it.
Use when you need global information before you can make local decisions.

> LeetCode: [217. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/), [219. Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/)

```csharp
// Contains Duplicate II — duplicate within distance k
bool ContainsNearbyDuplicate(int[] nums, int k)
{
    var map = new Dictionary<int, int>(); // val → last index
    for (int i = 0; i < nums.Length; i++)
    {
        if (map.TryGetValue(nums[i], out int prev) && i - prev <= k) return true;
        map[nums[i]] = i;
    }
    return false;
}
```

---

## Decision Guide

```
Do you need to count occurrences?
  └─ Yes → frequency map: map[x] = map.GetValueOrDefault(x, 0) + 1

Do you need to check if something exists?
  └─ No count needed → HashSet
  └─ Count needed    → Dictionary

Do you need to find a pair/complement?
  └─ a + b = target? → store seen values, check (target - current)
  └─ Array is sorted? → avoid HashMap, use two pointers instead (O(1) space)

Do you need the sum of a subarray?
  └─ Exact sum = k?         → prefix sum + HashMap
  └─ Longest subarray?      → prefix sum + first-seen index map
  └─ Array has only non-negatives + no exact target? → sliding window is simpler

Do you have a sliding window?
  └─ Track window contents? → HashMap of counts inside window
  └─ Only need existence, not count? → HashSet is enough

Do you need to group elements?
  └─ By a computed property? → map[computedKey].Add(element)
  └─ Need sorted groups? → avoid HashMap, use SortedDictionary

Do you need to find a sequence?
  └─ Consecutive numbers?    → HashSet + start-of-sequence trick
  └─ Need ordered traversal? → HashMap loses insertion order, use List or SortedDictionary
```

---

## Complexity Reference

| Operation | Average | Worst |
|---|---|---|
| Lookup / Insert / Delete | O(1) | O(n) — hash collision |
| Iteration over all pairs | O(n) | O(n) |
| Space | O(n) | O(n) |

---

## Common Mistakes

1. **KeyNotFoundException** — always use `TryGetValue` or `GetValueOrDefault` instead of `map[key]` when key might not exist.
2. **Mutating the map while iterating** — collect keys to remove first, then delete.
3. **Using a list as a key** — lists aren't hashable by value. Sort + join to a string, or use a tuple instead.
4. **Off-by-one in prefix sums** — always seed `map[0] = 1` before the loop.
5. **HashSet vs Dictionary confusion** — if you only need existence, HashSet is cleaner and slightly faster.