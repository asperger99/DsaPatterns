# Dynamic Programming — DSA Problem Solving Notes

## What is DP?

DP solves problems by breaking them into overlapping subproblems, solving each once, and storing the result so you never recompute it.

**Two signals that DP applies:**
1. **Optimal substructure** — the optimal answer contains optimal answers to subproblems
2. **Overlapping subproblems** — the same subproblem appears more than once in a naive recursion

**Core intuition:** write the plain recursion first. If you see the same call made multiple times, add a dp array. That's top-down DP.

---

## The Top-Down Template

```csharp
int[] dp = new int[n + 1];
Array.Fill(dp, -1); // -1 = not computed yet

int Solve(int i)
{
    if (/* base case */) return /* base value */;
    if (dp[i] != -1) return dp[i];
    return dp[i] = /* recurrence */;
}
```

**For 2D state:**
```csharp
int[,] dp = new int[m, n];
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        dp[i, j] = -1;

int Solve(int i, int j)
{
    if (/* base case */) return /* base value */;
    if (dp[i, j] != -1) return dp[i, j];
    return dp[i, j] = /* recurrence */;
}
```

**Initialization value:**
```
-1       → when result is always >= 0 (most common)
int.MinValue → when result can be 0 (and 0 is valid)
Use a separate bool[,] visited if result range overlaps with sentinel
```

---

## How to Approach Any DP Problem

```
1. Define: what does Solve(i) or Solve(i, j) return?
2. Recurrence: how does Solve(i) relate to Solve(i-1), Solve(i+1), etc.?
3. Base cases: smallest inputs you can answer directly
4. Size the dp array to fit all states
5. Trace a small example to verify
```

---

## Core Patterns

### 1. Linear DP — 1D

> LeetCode: [70. Climbing Stairs](https://leetcode.com/problems/climbing-stairs/), [198. House Robber](https://leetcode.com/problems/house-robber/), [213. House Robber II](https://leetcode.com/problems/house-robber-ii/)

```csharp
// Climbing Stairs — ways to reach step n (1 or 2 steps at a time)
// Solve(i) = number of ways to reach step i
int ClimbStairs(int n)
{
    var dp = new int[n + 1];
    Array.Fill(dp, -1);

    int Solve(int i)
    {
        if (i <= 1) return 1;
        if (dp[i] != -1) return dp[i];
        return dp[i] = Solve(i - 1) + Solve(i - 2);
    }

    return Solve(n);
}
```

```csharp
// House Robber — max money without robbing adjacent houses
// Solve(i) = max money from houses i..n-1
int Rob(int[] nums)
{
    var dp = new int[nums.Length];
    Array.Fill(dp, -1);

    int Solve(int i)
    {
        if (i >= nums.Length) return 0;
        if (dp[i] != -1) return dp[i];
        return dp[i] = Math.Max(
            Solve(i + 1),              // skip
            nums[i] + Solve(i + 2)    // rob
        );
    }

    return Solve(0);
}
```

```csharp
// House Robber II — houses in a circle
// Split into [0..n-2] and [1..n-1], take max
int RobII(int[] nums)
{
    if (nums.Length == 1) return nums[0];
    return Math.Max(RobRange(nums, 0, nums.Length - 2),
                    RobRange(nums, 1, nums.Length - 1));
}

int RobRange(int[] nums, int l, int r)
{
    var dp = new int[nums.Length];
    Array.Fill(dp, -1);

    int Solve(int i)
    {
        if (i > r) return 0;
        if (dp[i] != -1) return dp[i];
        return dp[i] = Math.Max(Solve(i + 1), nums[i] + Solve(i + 2));
    }

    return Solve(l);
}
```

---

### 2. Knapsack

> LeetCode: [416. Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/), [322. Coin Change](https://leetcode.com/problems/coin-change/), [518. Coin Change II](https://leetcode.com/problems/coin-change-ii/)

#### 0/1 Knapsack — each item used at most once

```csharp
// Solve(i, cap) = max value using items i..n-1 with remaining capacity
int Knapsack(int[] weights, int[] values, int cap)
{
    int n = weights.Length;
    var dp = new int[n, cap + 1];
    for (int i = 0; i < n; i++)
        for (int j = 0; j <= cap; j++)
            dp[i, j] = -1;

    int Solve(int i, int rem)
    {
        if (i == n || rem == 0) return 0;
        if (dp[i, rem] != -1) return dp[i, rem];

        int skip = Solve(i + 1, rem);
        int take = weights[i] <= rem ? values[i] + Solve(i + 1, rem - weights[i]) : 0;

        return dp[i, rem] = Math.Max(skip, take);
    }

    return Solve(0, cap);
}
```

#### Unbounded Knapsack — each item used unlimited times

```csharp
// Coin Change — min coins to make amount
// Solve(rem) = min coins to make rem
int CoinChange(int[] coins, int amount)
{
    var dp = new int[amount + 1];
    Array.Fill(dp, -1);

    int Solve(int rem)
    {
        if (rem == 0) return 0;
        if (rem < 0) return int.MaxValue;
        if (dp[rem] != -1) return dp[rem];

        int best = int.MaxValue;
        foreach (int c in coins)
        {
            int sub = Solve(rem - c);
            if (sub != int.MaxValue)
                best = Math.Min(best, sub + 1);
        }
        return dp[rem] = best;
    }

    int res = Solve(amount);
    return res == int.MaxValue ? -1 : res;
}
```

**Trace: coins=[1,2,5], amount=5**
```
Solve(5) → tries coin 5 → Solve(0)=0 → returns 1  ✓
```

```csharp
// Partition Equal Subset Sum
// Solve(i, rem) = can we pick from nums[i..] to sum exactly rem?
// Use int dp: -1=unvisited, 0=false, 1=true
bool CanPartition(int[] nums)
{
    int total = nums.Sum();
    if (total % 2 != 0) return false;
    int target = total / 2;

    var dp = new int[nums.Length, target + 1];
    for (int i = 0; i < nums.Length; i++)
        for (int j = 0; j <= target; j++)
            dp[i, j] = -1;

    int Solve(int i, int rem)
    {
        if (rem == 0) return 1;
        if (i == nums.Length || rem < 0) return 0;
        if (dp[i, rem] != -1) return dp[i, rem];
        return dp[i, rem] = Solve(i + 1, rem) == 1 || Solve(i + 1, rem - nums[i]) == 1 ? 1 : 0;
    }

    return Solve(0, target) == 1;
}
```

---

### 3. Longest Common Subsequence (LCS)

> LeetCode: [1143. LCS](https://leetcode.com/problems/longest-common-subsequence/), [72. Edit Distance](https://leetcode.com/problems/edit-distance/)

```csharp
// Solve(i, j) = LCS length of s1[i..] and s2[j..]
int LCS(string s1, string s2)
{
    int m = s1.Length, n = s2.Length;
    var dp = new int[m, n];
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            dp[i, j] = -1;

    int Solve(int i, int j)
    {
        if (i == m || j == n) return 0;
        if (dp[i, j] != -1) return dp[i, j];

        return dp[i, j] = s1[i] == s2[j]
            ? 1 + Solve(i + 1, j + 1)
            : Math.Max(Solve(i + 1, j), Solve(i, j + 1));
    }

    return Solve(0, 0);
}
```

```csharp
// Edit Distance
// Solve(i, j) = edit distance between s1[i..] and s2[j..]
int MinDistance(string s1, string s2)
{
    int m = s1.Length, n = s2.Length;
    var dp = new int[m + 1, n + 1];
    for (int i = 0; i <= m; i++)
        for (int j = 0; j <= n; j++)
            dp[i, j] = -1;

    int Solve(int i, int j)
    {
        if (i == m) return n - j; // insert remaining
        if (j == n) return m - i; // delete remaining
        if (dp[i, j] != -1) return dp[i, j];

        if (s1[i] == s2[j]) return dp[i, j] = Solve(i + 1, j + 1);

        return dp[i, j] = 1 + Math.Min(Solve(i + 1, j + 1),   // replace
                                Math.Min(Solve(i + 1, j),         // delete
                                         Solve(i, j + 1)));       // insert
    }

    return Solve(0, 0);
}
```

**Recurrence intuition:**
```
s1[i] == s2[j] → match, no cost → move both (i+1, j+1)
s1[i] != s2[j] → try all 3 ops, take min:
  replace → (i+1, j+1)
  delete  → (i+1, j)
  insert  → (i,   j+1)
```

---

### 4. Longest Increasing Subsequence (LIS)

> LeetCode: [300. LIS](https://leetcode.com/problems/longest-increasing-subsequence/)

```csharp
// Solve(i) = LIS length starting at index i
int LengthOfLIS(int[] nums)
{
    int n = nums.Length;
    var dp = new int[n];
    Array.Fill(dp, -1);

    int Solve(int i)
    {
        if (dp[i] != -1) return dp[i];
        int best = 1;
        for (int j = i + 1; j < n; j++)
            if (nums[j] > nums[i])
                best = Math.Max(best, 1 + Solve(j));
        return dp[i] = best;
    }

    int res = 1;
    for (int i = 0; i < n; i++)
        res = Math.Max(res, Solve(i));
    return res;
}
```

---

### 5. Grid DP

> LeetCode: [62. Unique Paths](https://leetcode.com/problems/unique-paths/), [64. Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/)

```csharp
// Minimum Path Sum
// Solve(r, c) = min cost to reach bottom-right from (r, c)
int MinPathSum(int[][] grid)
{
    int m = grid.Length, n = grid[0].Length;
    var dp = new int[m, n];
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            dp[i, j] = -1;

    int Solve(int r, int c)
    {
        if (r == m - 1 && c == n - 1) return grid[r][c];
        if (dp[r, c] != -1) return dp[r, c];

        int down  = r + 1 < m ? Solve(r + 1, c) : int.MaxValue;
        int right = c + 1 < n ? Solve(r, c + 1) : int.MaxValue;

        return dp[r, c] = grid[r][c] + Math.Min(down, right);
    }

    return Solve(0, 0);
}
```

```csharp
// Unique Paths II — with obstacles
int UniquePathsWithObstacles(int[][] grid)
{
    int m = grid.Length, n = grid[0].Length;
    var dp = new int[m, n];
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            dp[i, j] = -1;

    int Solve(int r, int c)
    {
        if (r >= m || c >= n || grid[r][c] == 1) return 0;
        if (r == m - 1 && c == n - 1) return 1;
        if (dp[r, c] != -1) return dp[r, c];
        return dp[r, c] = Solve(r + 1, c) + Solve(r, c + 1);
    }

    return Solve(0, 0);
}
```

---

### 6. State Machine DP — Stock Problems

> LeetCode: [121. Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/), [309. With Cooldown](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/), [188. Stock IV](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/)

```csharp
// Stock with at most k transactions
// Solve(i, txns, holding) = max profit from day i
// holding: 0=not holding, 1=holding
int MaxProfit(int k, int[] prices)
{
    int n = prices.Length;
    // dp[i][txns][holding]
    var dp = new int[n, k + 1, 2];
    for (int i = 0; i < n; i++)
        for (int t = 0; t <= k; t++)
            { dp[i, t, 0] = -1; dp[i, t, 1] = -1; }

    int Solve(int i, int txns, int holding)
    {
        if (i == n || txns == 0) return 0;
        if (dp[i, txns, holding] != -1) return dp[i, txns, holding];

        int doNothing = Solve(i + 1, txns, holding);
        int doSomething = holding == 1
            ? prices[i] + Solve(i + 1, txns - 1, 0)  // sell
            : -prices[i] + Solve(i + 1, txns, 1);     // buy

        return dp[i, txns, holding] = Math.Max(doNothing, doSomething);
    }

    return Solve(0, k, 0);
}
```

```csharp
// Stock with Cooldown
// Solve(i, holding) = max profit from day i
int MaxProfitCooldown(int[] prices)
{
    int n = prices.Length;
    var dp = new int[n, 2];
    for (int i = 0; i < n; i++) { dp[i, 0] = -1; dp[i, 1] = -1; }

    int Solve(int i, int holding)
    {
        if (i >= n) return 0;
        if (dp[i, holding] != -1) return dp[i, holding];

        int doNothing = Solve(i + 1, holding);
        int doSomething = holding == 1
            ? prices[i] + Solve(i + 2, 0)   // sell → cooldown skips i+1
            : -prices[i] + Solve(i + 1, 1); // buy

        return dp[i, holding] = Math.Max(doNothing, doSomething);
    }

    return Solve(0, 0);
}
```

**State transitions:**
```
holding=0, do nothing → Solve(i+1, 0)
holding=0, buy        → -price[i] + Solve(i+1, 1)
holding=1, do nothing → Solve(i+1, 1)
holding=1, sell       → +price[i] + Solve(i+2, 0)  ← i+2 if cooldown
```

---

### 7. Interval DP

> LeetCode: [516. Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/), [312. Burst Balloons](https://leetcode.com/problems/burst-balloons/)

```csharp
// Longest Palindromic Subsequence
// Solve(i, j) = LPS length in s[i..j]
int LongestPalindromeSubseq(string s)
{
    int n = s.Length;
    var dp = new int[n, n];
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            dp[i, j] = -1;

    int Solve(int i, int j)
    {
        if (i == j) return 1;
        if (i > j) return 0;
        if (dp[i, j] != -1) return dp[i, j];

        return dp[i, j] = s[i] == s[j]
            ? 2 + Solve(i + 1, j - 1)
            : Math.Max(Solve(i + 1, j), Solve(i, j - 1));
    }

    return Solve(0, n - 1);
}
```

---

### 8. String Partitioning

> LeetCode: [139. Word Break](https://leetcode.com/problems/word-break/), [132. Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/)

```csharp
// Word Break
// Solve(i) = can s[i..] be segmented? returns 0/1
bool WordBreak(string s, IList<string> wordDict)
{
    var words = new HashSet<string>(wordDict);
    var dp = new int[s.Length + 1]; // -1=unvisited, 0=false, 1=true
    Array.Fill(dp, -1);

    int Solve(int i)
    {
        if (i == s.Length) return 1;
        if (dp[i] != -1) return dp[i];

        for (int j = i + 1; j <= s.Length; j++)
            if (words.Contains(s.Substring(i, j - i)) && Solve(j) == 1)
                return dp[i] = 1;

        return dp[i] = 0;
    }

    return Solve(0) == 1;
}
```

---

## Sizing the Memo Array

```
Solve(i)           → int[n]
Solve(i, j)        → int[m, n]
Solve(i, rem)      → int[n, cap+1]
Solve(i, j, k)     → int[n, m, k]
Solve(i, holding)  → int[n, 2]

Rule: one dimension per parameter, sized by its range.
      always +1 if the param can equal the length (i == n as base case).
```

---

## Identifying the Pattern

```
Input is 1D, answer depends on previous choices?
  └─ Linear DP: Solve(i) → Solve(i±1), Solve(i±2)

Two sequences to compare?
  └─ LCS-style: Solve(i, j) over both

Single sequence, longest valid subsequence?
  └─ LIS: Solve(i) = best ending/starting at i, loop over j

Choose items within a budget?
  └─ Knapsack: Solve(i, rem) — take or skip
  └─ Each item once? → i+1 after taking
  └─ Reuse? → stay at i after taking (unbounded)

Grid traversal?
  └─ Solve(r, c) → Solve(r+1, c) and Solve(r, c+1)

Buy/sell with states?
  └─ Solve(i, holding) — add more params for cooldown/k transactions

Range [i, j] problem?
  └─ Interval DP: Solve(i, j) splits into sub-ranges

Count of ways?        → strong DP signal
Min/Max over all?     → strong DP signal

AVOID DP (use greedy) when:
  └─ Local optimal = global optimal

AVOID DP (use backtracking) when:
  └─ Need ALL solutions, not just count/value
```

---

## Common Mistakes

1. **Wrong state definition** — if the recurrence feels off, you're missing a state param. Add index, remaining capacity, holding flag, etc.
2. **Sentinel value clash** — using `-1` as "unvisited" but result can be `-1`. Use `int.MinValue` or a separate `bool[] visited` in that case.
3. **Memo array too small** — if param `i` reaches `n` as a base case, size the array `n+1`, not `n`.
4. **Forgetting the `return` on cache hit** — `if (dp[i] != -1) return dp[i];` — easy to write the check but forget the return.
5. **LIS final answer** — `Solve(i)` gives LIS starting at `i`. Final answer is `max over all i`, not just `Solve(0)`.
6. **Bool problems need int dp array** — C# arrays can't hold `-1` as bool. Use `int[]` with `0/1` for false/true and `-1` for unvisited.

---

## Complexity Reference

| Pattern | Time | Space (dp) |
|---|---|---|
| Linear DP | O(n) | O(n) |
| 0/1 Knapsack | O(n × cap) | O(n × cap) |
| LCS / Edit Distance | O(m × n) | O(m × n) |
| LIS | O(n²) | O(n) |
| Grid DP | O(m × n) | O(m × n) |
| State machine (stocks) | O(n × k × 2) | O(n × k × 2) |
| Interval DP | O(n³) | O(n²) |
| Word Break | O(n²) | O(n) |

If dp space is too large, switch to bottom-up with a rolling array.
