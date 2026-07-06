# Backtracking — DSA Problem Solving Notes

## What is Backtracking?

Backtracking is a systematic way to try all possibilities by building a solution incrementally and **undoing** (backtracking) a choice when it leads to a dead end.

**Core intuition:** think of it as a decision tree. At each node you make a choice, recurse into it, then undo that choice and try the next one. You're doing a DFS over all possible solutions and pruning branches early when they can't possibly lead to a valid answer.

```
Problem: find all subsets of [1, 2, 3]

Decision tree:
                    []
           /        |        \
         [1]       [2]       [3]
        /   \        \
     [1,2] [1,3]   [2,3]
      /
  [1,2,3]
```

---

## The Template

Every backtracking problem fits this shape:

```csharp
void Backtrack(/* state */, /* choices */)
{
    if (/* base case — solution complete */)
    {
        results.Add(new List<int>(current)); // snapshot, not reference
        return;
    }

    for (int i = start; i < choices.Length; i++)
    {
        if (/* pruning condition */) continue; // skip invalid choices early

        current.Add(choices[i]);   // 1. make choice
        Backtrack(i + 1, choices); // 2. recurse
        current.RemoveAt(current.Count - 1); // 3. undo choice
    }
}
```

**The three steps inside the loop are always:**
1. Make a choice (add to current state)
2. Recurse
3. Undo the choice (restore state exactly as it was)

---

## Core Patterns

### 1. Subsets

Generate all possible subsets (power set). Each element is either included or not.

> LeetCode: [78. Subsets](https://leetcode.com/problems/subsets/), [90. Subsets II](https://leetcode.com/problems/subsets-ii/)

```csharp
// Subsets — no duplicates in input
IList<IList<int>> Subsets(int[] nums)
{
    var res = new List<IList<int>>();
    var curr = new List<int>();

    void Backtrack(int start)
    {
        res.Add(new List<int>(curr)); // add at every node, not just leaves

        for (int i = start; i < nums.Length; i++)
        {
            curr.Add(nums[i]);
            Backtrack(i + 1);
            curr.RemoveAt(curr.Count - 1);
        }
    }

    Backtrack(0);
    return res;
}
```

```csharp
// Subsets II — input has duplicates, no duplicate subsets in output
IList<IList<int>> SubsetsWithDup(int[] nums)
{
    Array.Sort(nums); // sort so duplicates are adjacent
    var res = new List<IList<int>>();
    var curr = new List<int>();

    void Backtrack(int start)
    {
        res.Add(new List<int>(curr));

        for (int i = start; i < nums.Length; i++)
        {
            if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicate branches
            curr.Add(nums[i]);
            Backtrack(i + 1);
            curr.RemoveAt(curr.Count - 1);
        }
    }

    Backtrack(0);
    return res;
}
```

---

### 2. Combinations

Choose exactly `k` elements from `n`. Unlike subsets, you stop at a fixed size.

> LeetCode: [77. Combinations](https://leetcode.com/problems/combinations/), [39. Combination Sum](https://leetcode.com/problems/combination-sum/), [40. Combination Sum II](https://leetcode.com/problems/combination-sum-ii/)

```csharp
// Combinations — choose k from [1..n]
IList<IList<int>> Combine(int n, int k)
{
    var res = new List<IList<int>>();
    var curr = new List<int>();

    void Backtrack(int start)
    {
        if (curr.Count == k) { res.Add(new List<int>(curr)); return; }

        // pruning: not enough numbers left to fill k slots
        for (int i = start; i <= n - (k - curr.Count) + 1; i++)
        {
            curr.Add(i);
            Backtrack(i + 1);
            curr.RemoveAt(curr.Count - 1);
        }
    }

    Backtrack(1);
    return res;
}
```

```csharp
// Combination Sum — reuse elements, find all combos that sum to target
IList<IList<int>> CombinationSum(int[] nums, int target)
{
    var res = new List<IList<int>>();
    var curr = new List<int>();

    void Backtrack(int start, int remaining)
    {
        if (remaining == 0) { res.Add(new List<int>(curr)); return; }
        if (remaining < 0) return; // pruning

        for (int i = start; i < nums.Length; i++)
        {
            curr.Add(nums[i]);
            Backtrack(i, remaining - nums[i]); // i not i+1 → allows reuse
            curr.RemoveAt(curr.Count - 1);
        }
    }

    Backtrack(0, target);
    return res;
}
```

```csharp
// Combination Sum II — each element used once, no duplicate combos
IList<IList<int>> CombinationSum2(int[] nums, int target)
{
    Array.Sort(nums);
    var res = new List<IList<int>>();
    var curr = new List<int>();

    void Backtrack(int start, int remaining)
    {
        if (remaining == 0) { res.Add(new List<int>(curr)); return; }

        for (int i = start; i < nums.Length; i++)
        {
            if (nums[i] > remaining) break; // pruning — sorted, no point continuing
            if (i > start && nums[i] == nums[i - 1]) continue; // skip duplicates
            curr.Add(nums[i]);
            Backtrack(i + 1, remaining - nums[i]);
            curr.RemoveAt(curr.Count - 1);
        }
    }

    Backtrack(0, target);
    return res;
}
```

---

### 3. Permutations

Arrange all elements in every possible order. Unlike combinations, order matters.

> LeetCode: [46. Permutations](https://leetcode.com/problems/permutations/), [47. Permutations II](https://leetcode.com/problems/permutations-ii/)

```csharp
// Permutations — no duplicates in input
IList<IList<int>> Permute(int[] nums)
{
    var res = new List<IList<int>>();
    var curr = new List<int>();
    var used = new bool[nums.Length];

    void Backtrack()
    {
        if (curr.Count == nums.Length) { res.Add(new List<int>(curr)); return; }

        for (int i = 0; i < nums.Length; i++)
        {
            if (used[i]) continue;
            used[i] = true;
            curr.Add(nums[i]);
            Backtrack();
            curr.RemoveAt(curr.Count - 1);
            used[i] = false;
        }
    }

    Backtrack();
    return res;
}
```

```csharp
// Permutations II — input has duplicates
IList<IList<int>> PermuteUnique(int[] nums)
{
    Array.Sort(nums);
    var res = new List<IList<int>>();
    var curr = new List<int>();
    var used = new bool[nums.Length];

    void Backtrack()
    {
        if (curr.Count == nums.Length) { res.Add(new List<int>(curr)); return; }

        for (int i = 0; i < nums.Length; i++)
        {
            if (used[i]) continue;
            // skip duplicate: same value as previous AND previous wasn't used in this path
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
            used[i] = true;
            curr.Add(nums[i]);
            Backtrack();
            curr.RemoveAt(curr.Count - 1);
            used[i] = false;
        }
    }

    Backtrack();
    return res;
}
```

**Subsets vs Combinations vs Permutations:**
```
Subsets:      order doesn't matter, any size,   no reuse → pass i+1, add at every node
Combinations: order doesn't matter, fixed size, no reuse → pass i+1, add at leaf
Permutations: order matters,        fixed size, no reuse → pass used[], always start from 0
Combo Sum:    order doesn't matter, any size,   reuse    → pass i (not i+1)
```

---

### 4. Grid / Path Problems

Explore all paths in a grid. Use the grid itself to mark visited cells (restore after recursion).

> LeetCode: [79. Word Search](https://leetcode.com/problems/word-search/), [212. Word Search II](https://leetcode.com/problems/word-search-ii/)

```csharp
// Word Search — does word exist as a path in the grid?
bool Exist(char[][] board, string word)
{
    int m = board.Length, n = board[0].Length;
    int[][] dirs = [[0,1],[0,-1],[1,0],[-1,0]];

    bool Backtrack(int r, int c, int idx)
    {
        if (idx == word.Length) return true;
        if (r < 0 || r >= m || c < 0 || c >= n) return false;
        if (board[r][c] != word[idx]) return false;

        char tmp = board[r][c];
        board[r][c] = '#'; // mark visited (in-place, no extra array)

        bool found = false;
        foreach (var d in dirs)
            if (Backtrack(r + d[0], c + d[1], idx + 1)) { found = true; break; }

        board[r][c] = tmp; // restore
        return found;
    }

    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            if (Backtrack(r, c, 0)) return true;

    return false;
}
```

---

### 5. Constraint Satisfaction (N-Queens, Sudoku)

Place elements one at a time, check constraints before placing, undo if violated.

> LeetCode: [51. N-Queens](https://leetcode.com/problems/n-queens/), [37. Sudoku Solver](https://leetcode.com/problems/sudoku-solver/)

```csharp
// N-Queens
IList<IList<string>> SolveNQueens(int n)
{
    var res = new List<IList<string>>();
    var cols = new HashSet<int>();
    var diag1 = new HashSet<int>(); // r - c is same for each top-left diagonal
    var diag2 = new HashSet<int>(); // r + c is same for each top-right diagonal
    var board = new int[n]; // board[r] = column of queen in row r

    void Backtrack(int row)
    {
        if (row == n)
        {
            res.Add(board.Select(c => new string('.', c) + 'Q' + new string('.', n - c - 1)).ToList());
            return;
        }

        for (int col = 0; col < n; col++)
        {
            if (cols.Contains(col) || diag1.Contains(row - col) || diag2.Contains(row + col))
                continue; // queen attacks here — skip

            cols.Add(col); diag1.Add(row - col); diag2.Add(row + col);
            board[row] = col;

            Backtrack(row + 1);

            cols.Remove(col); diag1.Remove(row - col); diag2.Remove(row + col);
        }
    }

    Backtrack(0);
    return res;
}
```

---

### 6. String / Partition Problems

> LeetCode: [131. Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/), [93. Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/)

```csharp
// Palindrome Partitioning
IList<IList<string>> Partition(string s)
{
    var res = new List<IList<string>>();
    var curr = new List<string>();

    void Backtrack(int start)
    {
        if (start == s.Length) { res.Add(new List<string>(curr)); return; }

        for (int end = start + 1; end <= s.Length; end++)
        {
            string sub = s.Substring(start, end - start);
            if (!IsPalin(sub)) continue; // pruning — skip non-palindromes early

            curr.Add(sub);
            Backtrack(end);
            curr.RemoveAt(curr.Count - 1);
        }
    }

    Backtrack(0);
    return res;
}

bool IsPalin(string s)
{
    int l = 0, r = s.Length - 1;
    while (l < r) if (s[l++] != s[r--]) return false;
    return true;
}
```

---

## Pruning — The Key to Performance

Pruning means skipping entire branches of the decision tree early. It's what separates a correct backtracking solution from a fast one.

```
Common pruning conditions:

Remaining < 0 (combination sum)     → current path already exceeds target, stop
nums[i] > remaining (sorted input)  → all further elements also too big, break
curr.Count + remaining < k          → not enough elements left to reach size k
i > start && nums[i] == nums[i-1]  → duplicate value at same tree level, skip
board[r][c] already visited         → mark/unmark in-place
```

---

## Decision Guide

```
Generate ALL subsets?
  └─ Backtracking, add at every node (not just leaves)
  └─ Input has duplicates? → sort first, skip nums[i] == nums[i-1] at same level

Generate all combinations of size k?
  └─ Backtracking, add only at leaf (curr.Count == k)
  └─ Can reuse elements? → pass i instead of i+1 in recursive call

Generate all permutations?
  └─ Backtracking with used[] array, always loop from 0
  └─ Input has duplicates? → sort + skip if nums[i]==nums[i-1] && !used[i-1]

Path exists / find all paths in a grid?
  └─ Backtracking, mark cell visited in-place, restore after recursion
  └─ Just existence? → return true early on first valid path (don't collect all)

Place elements with constraints (N-Queens, Sudoku)?
  └─ Track constraint sets (cols, diagonals), add/remove around recursion

Partition a string into valid parts?
  └─ Backtracking over end index, prune invalid partitions early

AVOID backtracking when:
  └─ You only need COUNT of solutions → use DP instead (exponential → polynomial)
  └─ You need the OPTIMAL single solution → use DP or greedy
  └─ Problem has overlapping subproblems → memoized DFS or DP
```

---

## Common Mistakes

1. **Adding reference instead of snapshot** — `res.Add(curr)` adds a reference that will be modified. Always `res.Add(new List<int>(curr))`.
2. **Forgetting to undo** — every `curr.Add(...)` must have a matching `curr.RemoveAt(...)` after the recursive call, even in early-return paths.
3. **Reuse vs no-reuse** — passing `i` lets you reuse the current element; passing `i+1` moves past it. Mixing these up is the most common bug.
4. **Duplicate skipping condition wrong** — `i > start` (not `i > 0`) for combinations/subsets. Using `i > 0` skips valid first picks at deeper levels.
5. **Not sorting before dedup** — the duplicate-skip trick `nums[i] == nums[i-1]` only works if duplicates are adjacent. Always sort first.
6. **Collecting all paths when you only need one** — if the problem asks "does a path exist", return true immediately on the first hit instead of continuing.

---

## Complexity Reference

| Problem | Time | Space |
|---|---|---|
| Subsets | O(n × 2ⁿ) | O(n) |
| Combinations (n choose k) | O(k × C(n,k)) | O(k) |
| Permutations | O(n × n!) | O(n) |
| Combination Sum | O(n^(t/m)) t=target, m=min element | O(t/m) |
| N-Queens | O(n!) | O(n) |
| Word Search | O(m × n × 4^L) L=word length | O(L) |

All backtracking is exponential in the worst case. The win comes from pruning.