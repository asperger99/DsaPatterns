# Trie — DSA Problem Solving Notes

## What is a Trie?

A Trie (prefix tree) is a tree where each node represents a character, and paths from root to marked nodes spell out words. It's optimized for prefix-based operations on strings.

```
Words: ["cat", "can", "car", "dog"]

        root
       /    \
      c      d
      |      |
      a      o
     /|\     |
    t n r    g*
    *  * *

* = end of word
```

**Core intuition:** if a problem involves searching, inserting, or matching prefixes of strings, a Trie beats a HashSet. It collapses shared prefixes, saving space and enabling prefix queries in O(L) where L = word length.

---

## Trie Node & Implementation

```csharp
class TrieNode
{
    public TrieNode[] children = new TrieNode[26]; // for lowercase a-z
    public bool isEnd;
}

class Trie
{
    TrieNode root = new();

    public void Insert(string word)
    {
        var node = root;
        foreach (char c in word)
        {
            int i = c - 'a';
            node.children[i] ??= new TrieNode();
            node = node.children[i];
        }
        node.isEnd = true;
    }

    public bool Search(string word)
    {
        var node = Find(word);
        return node != null && node.isEnd;
    }

    public bool StartsWith(string prefix)
    {
        return Find(prefix) != null;
    }

    TrieNode Find(string s)
    {
        var node = root;
        foreach (char c in s)
        {
            int i = c - 'a';
            if (node.children[i] == null) return null;
            node = node.children[i];
        }
        return node;
    }
}
```

> LeetCode: [208. Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)

**All operations are O(L)** where L = length of the word/prefix. Independent of number of words stored.

---

## Core Patterns

### 1. Word Search with Wildcard

Support `.` as a wildcard that matches any character. DFS through the trie when you hit `.`.

> LeetCode: [211. Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/)

```csharp
class WordDictionary
{
    TrieNode root = new();

    public void AddWord(string word)
    {
        var node = root;
        foreach (char c in word)
        {
            int i = c - 'a';
            node.children[i] ??= new TrieNode();
            node = node.children[i];
        }
        node.isEnd = true;
    }

    public bool Search(string word) => DFS(word, 0, root);

    bool DFS(string word, int idx, TrieNode node)
    {
        if (idx == word.Length) return node.isEnd;

        char c = word[idx];
        if (c == '.')
        {
            // try all possible children
            foreach (var child in node.children)
                if (child != null && DFS(word, idx + 1, child)) return true;
            return false;
        }

        int i = c - 'a';
        return node.children[i] != null && DFS(word, idx + 1, node.children[i]);
    }
}
```

---

### 2. Word Search II — Find All Words in Grid

Build a Trie from the word list, then DFS on the grid. Prune branches when no word in the Trie starts with the current path.

> LeetCode: [212. Word Search II](https://leetcode.com/problems/word-search-ii/)

```csharp
IList<string> FindWords(char[][] board, string[] words)
{
    var root = new TrieNode();
    foreach (var w in words) Insert(root, w);

    int m = board.Length, n = board[0].Length;
    var res = new List<string>();
    int[][] dirs = [[0,1],[0,-1],[1,0],[-1,0]];

    void DFS(int r, int c, TrieNode node, string path)
    {
        if (r < 0 || r >= m || c < 0 || c >= n || board[r][c] == '#') return;

        char ch = board[r][c];
        var next = node.children[ch - 'a'];
        if (next == null) return; // no word starts with this path — prune

        path += ch;
        if (next.isEnd) { res.Add(path); next.isEnd = false; } // avoid duplicates

        board[r][c] = '#'; // mark visited
        foreach (var d in dirs) DFS(r + d[0], c + d[1], next, path);
        board[r][c] = ch;  // restore
    }

    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            DFS(r, c, root, "");

    return res;
}

void Insert(TrieNode root, string word)
{
    var node = root;
    foreach (char c in word)
    {
        int i = c - 'a';
        node.children[i] ??= new TrieNode();
        node = node.children[i];
    }
    node.isEnd = true;
}
```

**Why Trie beats HashSet here:**
With a HashSet you can't prune early — you must explore the full path before checking. With a Trie, the moment a grid path doesn't match any Trie prefix, you stop. This turns O(m×n×4^L) into something much faster in practice.

---

### 3. Longest Word / Prefix Queries

> LeetCode: [720. Longest Word in Dictionary](https://leetcode.com/problems/longest-word-in-dictionary/), [1268. Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/)

```csharp
// Longest word built one character at a time (every prefix is also a word)
string LongestWord(string[] words)
{
    var trie = new Trie();
    foreach (var w in words) trie.Insert(w);

    string best = "";
    // BFS or DFS from root, only follow paths where every node is end-of-word
    void DFS(TrieNode node, string curr)
    {
        if (curr.Length > best.Length || (curr.Length == best.Length && curr.CompareTo(best) < 0))
            best = curr;

        for (int i = 0; i < 26; i++)
        {
            var child = node.children[i];
            if (child != null && child.isEnd)
                DFS(child, curr + (char)('a' + i));
        }
    }

    DFS(trie.root, "");
    return best;
}
```

```csharp
// Search Suggestions System — for each prefix of searchWord, return 3 lexicographic suggestions
IList<IList<string>> SuggestedProducts(string[] products, string searchWord)
{
    Array.Sort(products); // lex order so first 3 found = lex smallest 3
    var trie = new Trie();
    // store word index at end node — omitted for brevity, use list at each node

    // Simpler approach: sort + binary search
    var res = new List<IList<string>>();
    string prefix = "";

    foreach (char c in searchWord)
    {
        prefix += c;
        var suggestions = new List<string>();
        // binary search for prefix start
        int lo = LowerBound(products, prefix);
        for (int i = lo; i < Math.Min(lo + 3, products.Length); i++)
            if (products[i].StartsWith(prefix)) suggestions.Add(products[i]);
        res.Add(suggestions);
    }
    return res;
}

int LowerBound(string[] arr, string target)
{
    int lo = 0, hi = arr.Length;
    while (lo < hi)
    {
        int mid = lo + (hi - lo) / 2;
        if (string.Compare(arr[mid], target) < 0) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
```

---

### 4. Replace Words with Root (Prefix Replacement)

> LeetCode: [648. Replace Words](https://leetcode.com/problems/replace-words/)

```csharp
// Replace each word in sentence with its shortest root from dictionary
string ReplaceWords(IList<string> dictionary, string sentence)
{
    var trie = new Trie();
    foreach (var root in dictionary) trie.Insert(root);

    var words = sentence.Split(' ');
    for (int w = 0; w < words.Length; w++)
    {
        var node = trie.root;
        var sb = new System.Text.StringBuilder();

        foreach (char c in words[w])
        {
            int i = c - 'a';
            if (node.children[i] == null) break; // no root prefix — keep original
            sb.Append(c);
            node = node.children[i];
            if (node.isEnd) { words[w] = sb.ToString(); break; } // found root
        }
    }
    return string.Join(' ', words);
}
```

---

### 5. Palindrome Pairs using Trie

> LeetCode: [336. Palindrome Pairs](https://leetcode.com/problems/palindrome-pairs/)

Instead of checking all O(n²) pairs, insert reversed words into a Trie. For each word, search the Trie to find words that form a palindrome when concatenated.

*(Complex to implement fully — the key insight is: build Trie of reversed words, for each word walk the Trie and check if remaining suffix/prefix is a palindrome.)*

---

### 6. XOR Maximum (Bitwise Trie)

A Trie doesn't have to store characters — it can store bits. Used for XOR-based problems.

> LeetCode: [421. Maximum XOR of Two Numbers in an Array](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/)

```csharp
// Insert numbers bit by bit (MSB to LSB) into a binary trie
// For each number, greedily pick the opposite bit to maximize XOR
int FindMaximumXOR(int[] nums)
{
    // Binary trie node: children[0] = bit 0, children[1] = bit 1
    var root = new int[2][] { null, null };
    var nodes = new List<int[][]> { new int[2][] { null, null } };

    void Insert(int num)
    {
        var node = nodes[0];
        for (int i = 31; i >= 0; i--)
        {
            int bit = (num >> i) & 1;
            if (node[bit] == null)
            {
                node[bit] = new int[] { -1, -1 };
                nodes.Add(new int[2][] { null, null });
                // simplified — use class-based node for clarity
            }
        }
    }

    // Cleaner class-based version:
    var r = new BitTrieNode();
    foreach (int n in nums) BitInsert(r, n);

    int best = 0;
    foreach (int n in nums) best = Math.Max(best, BitQuery(r, n));
    return best;
}

class BitTrieNode { public BitTrieNode[] children = new BitTrieNode[2]; }

void BitInsert(BitTrieNode root, int num)
{
    var node = root;
    for (int i = 31; i >= 0; i--)
    {
        int bit = (num >> i) & 1;
        node.children[bit] ??= new BitTrieNode();
        node = node.children[bit];
    }
}

int BitQuery(BitTrieNode root, int num)
{
    var node = root;
    int xor = 0;
    for (int i = 31; i >= 0; i--)
    {
        int bit = (num >> i) & 1;
        int want = 1 - bit; // want opposite bit to maximize XOR
        if (node.children[want] != null) { xor |= (1 << i); node = node.children[want]; }
        else node = node.children[bit]; // opposite not available, take same
    }
    return xor;
}
```

**Trace: nums=[3,10,5,25,2,8], find max XOR**
```
For num=5 (00...00101):
  bit 31..3 = 0 → always take same (no choice)
  bit 2: want 1, have 1 (from 25=11001) → xor |= 4, go right
  bit 1: want 0, have 0 (from 25) → xor |= 2... 
  
  5 XOR 25 = 28  ← max XOR in array ✓
```

---

## Trie vs HashMap

```
Use HashMap when:
  └─ Exact word lookup (no prefix queries needed)
  └─ Storing arbitrary keys (not just strings)
  └─ Implementation simplicity matters

Use Trie when:
  └─ Prefix search / autocomplete
  └─ Wildcard matching
  └─ Finding all words with a given prefix
  └─ Shared prefix compression saves space (many words, long common prefixes)
  └─ XOR / bitwise optimization (binary trie)
```

---

## Decision Guide

```
Insert and search exact words + prefix queries?
  └─ Trie — O(L) per operation

Wildcard matching (. matches any char)?
  └─ Trie + DFS on wildcard nodes

Find all words in a grid?
  └─ Trie of word list + DFS on grid (prune via trie)

Prefix replacement (use shortest root)?
  └─ Trie — walk until isEnd, replace with path so far

Maximize XOR of two numbers?
  └─ Binary trie — insert all numbers, greedily query opposite bits

AVOID Trie when:
  └─ Only exact match needed → HashSet is simpler
  └─ Keys are not strings/bit sequences → use HashMap
  └─ Character set is large (unicode) → children array wastes space, use Dictionary<char, TrieNode>
  └─ Word list is tiny → HashSet + startsWith is fine
```

---

## Common Mistakes

1. **`children` array size** — `new TrieNode[26]` only works for lowercase a-z. For uppercase + lowercase use 52, for all ASCII use 128, or switch to `Dictionary<char, TrieNode>`.
2. **Checking `isEnd` too early** — `Search` must check `isEnd` on the final node, not along the path. `StartsWith` does NOT check `isEnd`.
3. **Not marking visited in Word Search II** — set `board[r][c] = '#'` before recursing, restore after. Without this you revisit cells.
4. **Duplicate results in Word Search II** — once a word is found, set `node.isEnd = false` to prevent adding it again from a different path.
5. **Using `string +=` in DFS** — creates a new string every step, O(L²) total. Use `StringBuilder` or pass index + reconstruct at leaf.
6. **Binary Trie bit order** — always insert from MSB (bit 31) to LSB (bit 0). Reversing this gives wrong XOR results.

---

## Complexity Reference

| Operation | Time | Space |
|---|---|---|
| Insert word | O(L) | O(L × 26) worst |
| Search word | O(L) | O(1) |
| StartsWith prefix | O(L) | O(1) |
| Word Search II | O(m×n×4^L) worst, much less with pruning | O(W×L) — W words |
| Max XOR (binary trie) | O(n × 32) | O(n × 32) |

L = average word length, W = number of words.