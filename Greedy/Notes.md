# Greedy — DSA Problem Solving Notes

## What is Greedy?

A greedy algorithm makes the locally optimal choice at each step, hoping it leads to a globally optimal solution. No backtracking — once a choice is made, it's final.

**Core intuition:** greedy works when a locally best choice never needs to be undone. The hard part is proving that the greedy choice is safe — usually via "exchange argument": if you swap the greedy choice with any other, the result is no better.

```
Greedy: always pick the best available option right now
DP:     consider all options, memoize overlapping subproblems
Backtracking: try all options, undo bad choices
```

**When greedy works:**
- Greedy choice property — a globally optimal solution includes the locally optimal choice
- Optimal substructure — optimal solution to the problem contains optimal solutions to subproblems

---

## Core Patterns

### 1. Interval Scheduling / Activity Selection

Sort by end time. Greedily pick intervals that don't overlap with the last chosen one. Maximizes the number of non-overlapping intervals.

> LeetCode: [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/), [452. Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/)

```csharp
// Minimum number of intervals to remove so no two overlap
int EraseOverlapIntervals(int[][] intervals)
{
    Array.Sort(intervals, (a, b) => a[1] - b[1]); // sort by end time
    int removed = 0, lastEnd = int.MinValue;

    foreach (var iv in intervals)
    {
        if (iv[0] >= lastEnd) lastEnd = iv[1]; // no overlap — keep it
        else removed++;                          // overlap — remove current (ends later)
    }
    return removed;
}
```

**Trace: [[1,2],[2,3],[3,4],[1,3]]**
```
Sort by end: [[1,2],[2,3],[1,3],[3,4]]

[1,2]: 1 >= MIN → keep, lastEnd=2
[2,3]: 2 >= 2  → keep, lastEnd=3
[1,3]: 1 < 3   → remove (+1)
[3,4]: 3 >= 3  → keep, lastEnd=4

removed = 1 ✓
```

**Why sort by end time, not start?**
Ending earliest leaves maximum room for future intervals. Ending latest unnecessarily blocks future choices.

```csharp
// Minimum arrows to burst all balloons (overlapping intervals share an arrow)
int FindMinArrowShots(int[][] points)
{
    Array.Sort(points, (a, b) => a[1].CompareTo(b[1])); // sort by end
    int arrows = 1, end = points[0][1];

    for (int i = 1; i < points.Length; i++)
    {
        if (points[i][0] > end) // balloon starts after current arrow
        {
            arrows++;
            end = points[i][1]; // shoot new arrow at end of this balloon
        }
        // else: same arrow bursts this balloon too
    }
    return arrows;
}
```

---

### 2. Jump Game

> LeetCode: [55. Jump Game](https://leetcode.com/problems/jump-game/), [45. Jump Game II](https://leetcode.com/problems/jump-game-ii/)

```csharp
// Can you reach the last index?
bool CanJump(int[] nums)
{
    int reach = 0;
    for (int i = 0; i < nums.Length; i++)
    {
        if (i > reach) return false;      // can't reach this position
        reach = Math.Max(reach, i + nums[i]); // update max reachable
    }
    return true;
}
```

```csharp
// Minimum jumps to reach the last index
int Jump(int[] nums)
{
    int jumps = 0, curEnd = 0, farthest = 0;

    for (int i = 0; i < nums.Length - 1; i++)
    {
        farthest = Math.Max(farthest, i + nums[i]); // best we can reach from this window
        if (i == curEnd) // reached end of current jump's range — must jump
        {
            jumps++;
            curEnd = farthest;
        }
    }
    return jumps;
}
```

**Trace: [2,3,1,1,4]**
```
i=0: farthest=max(0,0+2)=2, i==curEnd(0) → jump=1, curEnd=2
i=1: farthest=max(2,1+3)=4
i=2: farthest=max(4,2+1)=4, i==curEnd(2) → jump=2, curEnd=4
i=3: farthest=max(4,3+1)=4

jumps = 2 ✓
```

---

### 3. Gas Station (Circular Greedy)

> LeetCode: [134. Gas Station](https://leetcode.com/problems/gas-station/)

```csharp
// Can complete the circuit? If yes, find the starting station.
int CanCompleteCircuit(int[] gas, int[] cost)
{
    int total = 0, tank = 0, start = 0;

    for (int i = 0; i < gas.Length; i++)
    {
        int gain = gas[i] - cost[i];
        total += gain;
        tank  += gain;

        if (tank < 0) // can't continue from current start
        {
            start = i + 1; // try starting from the next station
            tank  = 0;
        }
    }

    return total >= 0 ? start : -1; // total < 0 means no solution exists
}
```

**Why it works:**
- If total gas >= total cost, a solution always exists
- If we run out at station `i`, no station between `start` and `i` can be the answer (they all had enough gas to get to `i` before running dry)
- So reset start to `i+1`

---

### 4. Task Scheduler

> LeetCode: [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/)

```csharp
// Minimum time to finish all tasks with cooldown n between same tasks
int LeastInterval(char[] tasks, int n)
{
    var freq = new int[26];
    foreach (char t in tasks) freq[t - 'A']++;
    Array.Sort(freq);

    int maxFreq = freq[25];
    int idleSlots = (maxFreq - 1) * n; // slots needed around the most frequent task

    // fill idle slots with other tasks (most frequent first)
    for (int i = 24; i >= 0 && idleSlots > 0; i--)
        idleSlots -= Math.Min(freq[i], maxFreq - 1);

    idleSlots = Math.Max(0, idleSlots); // can't have negative idle slots
    return tasks.Length + idleSlots;
}
```

**Trace: tasks=["A","A","A","B","B","B"], n=2**
```
freq: A=3, B=3
maxFreq = 3, idleSlots = (3-1)*2 = 4

Fill with B (freq=3, min(3, 2)=2): idleSlots = 4-2 = 2

tasks.Length + idleSlots = 6 + 2 = 8

Layout: A B _ A B _ A B  ✓ (8 slots)
```

---

### 5. Assign Cookies / Two-Array Greedy

Sort both arrays. Greedily match the smallest sufficient resource to the smallest need.

> LeetCode: [455. Assign Cookies](https://leetcode.com/problems/assign-cookies/), [881. Boats to Save People](https://leetcode.com/problems/boats-to-save-people/)

```csharp
// Assign cookies — maximize number of content children
int FindContentChildren(int[] g, int[] s) // g=greed, s=size
{
    Array.Sort(g);
    Array.Sort(s);
    int child = 0, cookie = 0;

    while (child < g.Length && cookie < s.Length)
    {
        if (s[cookie] >= g[child]) child++; // cookie satisfies child
        cookie++;                            // always move to next cookie
    }
    return child;
}
```

```csharp
// Boats — each boat holds at most 2 people with combined weight <= limit
int NumRescueBoats(int[] people, int limit)
{
    Array.Sort(people);
    int lo = 0, hi = people.Length - 1, boats = 0;

    while (lo <= hi)
    {
        if (people[lo] + people[hi] <= limit) lo++; // lightest fits with heaviest
        hi--;    // heaviest always gets a boat
        boats++;
    }
    return boats;
}
```

---

### 6. Candy / Greedy with Two Passes

Some greedy problems need two passes — one left-to-right, one right-to-left — then combine.

> LeetCode: [135. Candy](https://leetcode.com/problems/candy/)

```csharp
// Each child must have at least 1 candy.
// Children with higher rating than their neighbor get more candy.
int Candy(int[] ratings)
{
    int n = ratings.Length;
    var candies = new int[n];
    Array.Fill(candies, 1);

    // left pass: if rating increases left to right, give one more than left neighbor
    for (int i = 1; i < n; i++)
        if (ratings[i] > ratings[i - 1])
            candies[i] = candies[i - 1] + 1;

    // right pass: if rating increases right to left, take max of current and right+1
    for (int i = n - 2; i >= 0; i--)
        if (ratings[i] > ratings[i + 1])
            candies[i] = Math.Max(candies[i], candies[i + 1] + 1);

    return candies.Sum();
}
```

**Trace: ratings = [1, 0, 2]**
```
After left pass:  [1, 1, 2]
After right pass: [2, 1, 2]  (ratings[0]=1 > ratings[1]=0 → max(1, 1+1)=2)
Sum = 5 ✓
```

---

### 7. Largest Number (Custom Comparator Greedy)

> LeetCode: [179. Largest Number](https://leetcode.com/problems/largest-number/)

```csharp
string LargestNumber(int[] nums)
{
    var strs = nums.Select(n => n.ToString()).ToArray();
    Array.Sort(strs, (a, b) => string.Compare(b + a, a + b)); // greedy: prefer order that gives larger concatenation

    if (strs[0] == "0") return "0";
    return string.Concat(strs);
}
```

**Why compare `b+a` vs `a+b`:**
```
"9" vs "34": "934" > "349" → 9 comes first
"3" vs "30": "330" > "303" → 3 comes first
```

---

### 8. Stock Problems (Greedy for Unlimited Transactions)

> LeetCode: [122. Best Time to Buy and Sell Stock II](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/)

```csharp
// Unlimited transactions — collect every upward slope
int MaxProfit(int[] prices)
{
    int profit = 0;
    for (int i = 1; i < prices.Length; i++)
        if (prices[i] > prices[i - 1])
            profit += prices[i] - prices[i - 1]; // grab every gain
    return profit;
}
```

**Why this works:** holding stock from day 1 to day 5 with ups and downs = sum of all positive daily changes. Greedy captures every profitable move.

> Note: For 1 transaction, k transactions, or cooldown variants → use DP (see DynamicProgramming notes).

---

### 9. Partition Labels

> LeetCode: [763. Partition Labels](https://leetcode.com/problems/partition-labels/)

```csharp
// Partition string so each letter appears in at most one part, maximize parts
IList<int> PartitionLabels(string s)
{
    var last = new int[26];
    for (int i = 0; i < s.Length; i++) last[s[i] - 'a'] = i; // last occurrence of each char

    var res = new List<int>();
    int start = 0, end = 0;

    for (int i = 0; i < s.Length; i++)
    {
        end = Math.Max(end, last[s[i] - 'a']); // extend partition to cover this char's last occurrence
        if (i == end) // reached the end of a valid partition
        {
            res.Add(end - start + 1);
            start = i + 1;
        }
    }
    return res;
}
```

**Trace: s = "ababcbacadefegdehijhklij"**
```
last: a=8, b=5, c=7, d=14, e=15, f=11, g=13, h=19, i=22, j=23, k=20, l=21

i=0(a): end=max(0,8)=8
i=1(b): end=max(8,5)=8
...
i=8(a): end=8, i==end → partition size=9, start=9

i=9(d): end=max(0,14)=14
...
i=15(e): end=15, i==end → partition size=7, start=16

Result: [9, 7, 8] ✓
```

---

## Greedy vs DP

```
Use Greedy when:
  └─ Local optimal = global optimal (can prove by exchange argument)
  └─ Choices are independent — making one doesn't affect validity of others
  └─ Problem asks: min/max count, earliest/latest time, most/fewest something

Use DP when:
  └─ Choices interact — taking one option affects future options' costs
  └─ You need to consider multiple possibilities (not just local best)
  └─ Problem has overlapping subproblems

Examples:
  └─ Activity selection → Greedy (sort by end, pick earliest-ending non-overlapping)
  └─ 0/1 Knapsack → DP (including item affects remaining capacity)
  └─ Coin change (unlimited) → DP (local best coin may prevent optimal total)
  └─ Stock with unlimited trades → Greedy (all upward slopes, no dependency)
  └─ Stock with k trades → DP (choices interact)
```

---

## Decision Guide

```
Interval scheduling (max non-overlapping intervals)?
  └─ Sort by end time → greedy pick non-overlapping

Minimum resources to cover all intervals?
  └─ Sort by start, heap of end times (see Heap notes)

Reach the end of an array with jumps?
  └─ Track max reachable index — greedy expand window

Circular route feasibility?
  └─ Track total and running tank; reset start on negative tank

Task scheduling with cooldown?
  └─ Idle slots = (maxFreq-1) * n, fill with remaining tasks

Match two sorted arrays (cookies, boats)?
  └─ Two pointers after sorting — match smallest to smallest

Need to satisfy both left and right neighbors?
  └─ Two-pass greedy — left pass, then right pass, take max

Partition/group string with constraints?
  └─ Precompute last occurrence, extend partition on each character

AVOID Greedy when:
  └─ Making a local choice now changes costs later → DP
  └─ You need to track combinations/counts of solutions → DP/Backtracking
  └─ Choices are not independent → DP
  └─ Can construct a counterexample to the greedy strategy → DP
```

---

## Common Mistakes

1. **Wrong sort key for intervals** — merge intervals: sort by start. Non-overlapping / arrow problems: sort by end. Mixing them gives wrong greedy decisions.
2. **Greedy without proof** — greedy feels right but isn't always correct. Always try to find a counterexample. If you can't, it's likely correct.
3. **Task scheduler formula** — `idleSlots = (maxFreq - 1) * n` can go negative if there are enough other tasks. Clamp at 0 with `Math.Max(0, idleSlots)`.
4. **Gas station reset** — when tank goes negative, reset both `tank = 0` and `start = i + 1`. Forgetting to reset `tank` means you carry a debt forward.
5. **Jump Game II boundary** — loop only to `nums.Length - 1`, not `nums.Length`. At the last index you don't need to jump further.
6. **Two-pass greedy order** — in candy, the right pass must take `Math.Max` of current and right+1. Overwriting (not max) loses the left-pass constraint.

---

## Complexity Reference

| Problem | Time | Space |
|---|---|---|
| Interval scheduling (sort + scan) | O(n log n) | O(1) |
| Jump Game I / II | O(n) | O(1) |
| Gas Station | O(n) | O(1) |
| Task Scheduler | O(n) | O(1) — 26 char freq array |
| Assign Cookies / Boats | O(n log n) | O(1) |
| Candy (two-pass) | O(n) | O(n) |
| Partition Labels | O(n) | O(1) — 26 char last array |
| Largest Number | O(n log n) | O(n) |