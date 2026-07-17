# Trees — DSA Problem Solving Notes

## Core Definitions

**Binary Tree** — each node has at most 2 children (left, right). No ordering guarantee.
**BST** — binary tree where `left < node < right` for every node. Enables O(log n) search.
**Balanced Tree** — height is O(log n). AVL, Red-Black trees.
**Complete Tree** — all levels filled except possibly the last (filled left to right).
**Perfect Tree** — all levels completely filled.

```csharp
class TreeNode
{
    public int val;
    public TreeNode left, right;
    public TreeNode(int val) { this.val = val; }
}
```

---

## Traversals

### DFS — Recursive (most common in interviews)

```csharp
void Inorder(TreeNode node)   // left → root → right  (gives sorted order in BST)
{
    if (node == null) return;
    Inorder(node.left);
    Console.Write(node.val);
    Inorder(node.right);
}

void Preorder(TreeNode node)  // root → left → right  (used to clone/serialize trees)
{
    if (node == null) return;
    Console.Write(node.val);
    Preorder(node.left);
    Preorder(node.right);
}

void Postorder(TreeNode node) // left → right → root  (used to delete/process children first)
{
    if (node == null) return;
    Postorder(node.left);
    Postorder(node.right);
    Console.Write(node.val);
}
```

### DFS — Iterative (when recursion stack is a concern)

```csharp
IList<int> InorderIterative(TreeNode root)
{
    var res = new List<int>();
    var stack = new Stack<TreeNode>();
    var curr = root;

    while (curr != null || stack.Count > 0)
    {
        while (curr != null) { stack.Push(curr); curr = curr.left; } // go left
        curr = stack.Pop();
        res.Add(curr.val);   // process
        curr = curr.right;   // go right
    }
    return res;
}
```

### BFS — Level Order

```csharp
IList<IList<int>> LevelOrder(TreeNode root)
{
    var res = new List<IList<int>>();
    if (root == null) return res;

    var q = new Queue<TreeNode>();
    q.Enqueue(root);

    while (q.Count > 0)
    {
        int size = q.Count; // snapshot — number of nodes at this level
        var level = new List<int>();
        for (int i = 0; i < size; i++)
        {
            var node = q.Dequeue();
            level.Add(node.val);
            if (node.left != null) q.Enqueue(node.left);
            if (node.right != null) q.Enqueue(node.right);
        }
        res.Add(level);
    }
    return res;
}
```

**When to use which:**
```
Inorder    → BST problems (gives sorted sequence)
Preorder   → serialize/clone tree, construct from traversal
Postorder  → delete tree, compute subtree values bottom-up
Level order→ shortest path, level-by-level processing, right side view
```

---

## Core Patterns

### 1. Tree Height / Depth

> LeetCode: [104. Maximum Depth](https://leetcode.com/problems/maximum-depth-of-binary-tree/), [111. Minimum Depth](https://leetcode.com/problems/minimum-depth-of-binary-tree/)

```csharp
// Height = max depth from root to any leaf
int MaxDepth(TreeNode node)
{
    if (node == null) return 0;
    return 1 + Math.Max(MaxDepth(node.left), MaxDepth(node.right));
}
```

```csharp
// Min depth = shortest path root → leaf (leaf = no children)
int MinDepth(TreeNode node)
{
    if (node == null) return 0;
    if (node.left == null && node.right == null) return 1; // leaf

    if (node.left == null) return 1 + MinDepth(node.right); // only right child
    if (node.right == null) return 1 + MinDepth(node.left); // only left child

    return 1 + Math.Min(MinDepth(node.left), MinDepth(node.right));
}
```

**Min depth trap:** a node with one null child is NOT a leaf. If you just do `Math.Min(left, right)`, the null side returns 0 and you get the wrong answer.

---

### 2. Diameter and Path Problems

The diameter (or any path passing through a node) is computed as a **side effect** during height calculation.

> LeetCode: [543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/), [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/)

```csharp
// Diameter — longest path between any two nodes (may not pass through root)
int DiameterOfBinaryTree(TreeNode root)
{
    int best = 0;

    int Height(TreeNode node)
    {
        if (node == null) return 0;
        int l = Height(node.left);
        int r = Height(node.right);
        best = Math.Max(best, l + r); // diameter through this node
        return 1 + Math.Max(l, r);    // height returned to parent
    }

    Height(root);
    return best;
}
```

```csharp
// Max Path Sum — path can start/end anywhere
int MaxPathSum(TreeNode root)
{
    int best = int.MinValue;

    int Gain(TreeNode node)
    {
        if (node == null) return 0;
        int l = Math.Max(0, Gain(node.left));  // ignore negative paths
        int r = Math.Max(0, Gain(node.right));
        best = Math.Max(best, node.val + l + r); // path through this node
        return node.val + Math.Max(l, r);        // only one side returned up
    }

    Gain(root);
    return best;
}
```

**Pattern:** these problems have two return values — one for the **answer** (updated globally), one for the **value returned to parent**. They're different.

---

### 3. Balanced Tree Check

> LeetCode: [110. Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/)

```csharp
// A tree is balanced if every subtree has |left height - right height| <= 1
bool IsBalanced(TreeNode root)
{
    int Check(TreeNode node)
    {
        if (node == null) return 0;
        int l = Check(node.left);
        if (l == -1) return -1; // already unbalanced
        int r = Check(node.right);
        if (r == -1) return -1;
        if (Math.Abs(l - r) > 1) return -1; // unbalanced here
        return 1 + Math.Max(l, r);
    }

    return Check(root) != -1;
}
```

Use `-1` as a sentinel to short-circuit early. Don't check balance and height in separate passes — that's O(n²).

---

### 4. Same Tree / Symmetric / Mirror

> LeetCode: [100. Same Tree](https://leetcode.com/problems/same-tree/), [101. Symmetric Tree](https://leetcode.com/problems/symmetric-tree/)

```csharp
bool IsSameTree(TreeNode p, TreeNode q)
{
    if (p == null && q == null) return true;
    if (p == null || q == null) return false;
    return p.val == q.val && IsSameTree(p.left, q.left) && IsSameTree(p.right, q.right);
}
```

```csharp
bool IsSymmetric(TreeNode root)
{
    bool Mirror(TreeNode l, TreeNode r)
    {
        if (l == null && r == null) return true;
        if (l == null || r == null) return false;
        return l.val == r.val && Mirror(l.left, r.right) && Mirror(l.right, r.left);
    }
    return Mirror(root.left, root.right);
}
```

---

### 5. Lowest Common Ancestor (LCA)

> LeetCode: [236. LCA of Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/), [235. LCA of BST](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

```csharp
// General binary tree
TreeNode LCA(TreeNode root, TreeNode p, TreeNode q)
{
    if (root == null || root == p || root == q) return root;

    var left  = LCA(root.left, p, q);
    var right = LCA(root.right, p, q);

    if (left != null && right != null) return root; // p and q on different sides
    return left ?? right;                            // both on same side
}
```

```csharp
// BST — use ordering to navigate
TreeNode LCA_BST(TreeNode root, TreeNode p, TreeNode q)
{
    if (p.val < root.val && q.val < root.val) return LCA_BST(root.left, p, q);
    if (p.val > root.val && q.val > root.val) return LCA_BST(root.right, p, q);
    return root; // split point — this is the LCA
}
```

---

### 6. BST Operations

> LeetCode: [98. Validate BST](https://leetcode.com/problems/validate-binary-search-tree/), [700. Search in BST](https://leetcode.com/problems/search-in-a-binary-search-tree/), [450. Delete Node in BST](https://leetcode.com/problems/delete-node-in-a-bst/)

```csharp
// Validate BST — pass valid range down, don't just check parent
bool IsValidBST(TreeNode node, long min = long.MinValue, long max = long.MaxValue)
{
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return IsValidBST(node.left, min, node.val) &&
           IsValidBST(node.right, node.val, max);
}
```

**Common BST mistake:** only comparing a node to its direct parent misses cases like:
```
    5
   / \
  1   6
     / \
    3   7    ← 3 < 5 but is in right subtree of 5 → invalid
```
Always pass `min` and `max` bounds down.

```csharp
// Delete node in BST
TreeNode DeleteNode(TreeNode root, int key)
{
    if (root == null) return null;

    if (key < root.val) { root.left = DeleteNode(root.left, key); }
    else if (key > root.val) { root.right = DeleteNode(root.right, key); }
    else
    {
        if (root.left == null) return root.right;  // no left child
        if (root.right == null) return root.left;  // no right child

        // two children — replace with inorder successor (smallest in right subtree)
        var succ = root.right;
        while (succ.left != null) succ = succ.left;
        root.val = succ.val;
        root.right = DeleteNode(root.right, succ.val);
    }
    return root;
}
```

---

### 7. Tree Construction

> LeetCode: [105. Construct from Preorder and Inorder](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/), [108. Convert Sorted Array to BST](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/)

```csharp
// Build tree from preorder + inorder
TreeNode BuildTree(int[] preorder, int[] inorder)
{
    var indexMap = new Dictionary<int, int>();
    for (int i = 0; i < inorder.Length; i++) indexMap[inorder[i]] = i;

    int pre = 0;

    TreeNode Build(int l, int r)
    {
        if (l > r) return null;
        var node = new TreeNode(preorder[pre++]);  // next preorder = current root
        int mid = indexMap[node.val];              // find root in inorder
        node.left = Build(l, mid - 1);             // left subtree
        node.right = Build(mid + 1, r);            // right subtree
        return node;
    }

    return Build(0, inorder.Length - 1);
}
```

```csharp
// Sorted array → height-balanced BST
TreeNode SortedArrayToBST(int[] nums)
{
    TreeNode Build(int l, int r)
    {
        if (l > r) return null;
        int mid = l + (r - l) / 2;
        var node = new TreeNode(nums[mid]);
        node.left = Build(l, mid - 1);
        node.right = Build(mid + 1, r);
        return node;
    }
    return Build(0, nums.Length - 1);
}
```

---

### 8. Path Sum Problems

> LeetCode: [112. Path Sum](https://leetcode.com/problems/path-sum/), [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/), [437. Path Sum III](https://leetcode.com/problems/path-sum-iii/)

```csharp
// Path Sum — does root-to-leaf path with given sum exist?
bool HasPathSum(TreeNode node, int target)
{
    if (node == null) return false;
    if (node.left == null && node.right == null) return node.val == target; // leaf
    return HasPathSum(node.left, target - node.val) ||
           HasPathSum(node.right, target - node.val);
}
```

```csharp
// Path Sum III — count paths (any node to any node downward) summing to target
// Use prefix sum + hashmap at each root-to-node path
int PathSum(TreeNode root, int target)
{
    var prefixCount = new Dictionary<long, int>();
    prefixCount[0] = 1;

    int DFS(TreeNode node, long curr)
    {
        if (node == null) return 0;
        curr += node.val;
        int count = prefixCount.GetValueOrDefault(curr - target, 0);
        prefixCount[curr] = prefixCount.GetValueOrDefault(curr, 0) + 1;
        count += DFS(node.left, curr) + DFS(node.right, curr);
        prefixCount[curr]--; // backtrack — remove this node's prefix from map
        return count;
    }

    return DFS(root, 0);
}
```

---

### 9. Serialize / Deserialize

> LeetCode: [297. Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)

```csharp
// Preorder serialization — null nodes marked as "#"
string Serialize(TreeNode root)
{
    if (root == null) return "#";
    return $"{root.val},{Serialize(root.left)},{Serialize(root.right)}";
}

TreeNode Deserialize(string data)
{
    var vals = new Queue<string>(data.Split(','));

    TreeNode Build()
    {
        var v = vals.Dequeue();
        if (v == "#") return null;
        var node = new TreeNode(int.Parse(v));
        node.left = Build();
        node.right = Build();
        return node;
    }

    return Build();
}
```

---

## Key Insight: Return Value vs Global Update

Most tree problems that compute something **across** the tree (path, diameter, sum) follow this shape:

```csharp
int best; // global answer updated during traversal

int DFS(TreeNode node)
{
    if (node == null) return /* base */;

    int l = DFS(node.left);
    int r = DFS(node.right);

    best = Math.Max(best, /* combine l, r, node.val */); // update answer

    return /* value useful to parent — usually only one side */;
}
```

The **answer** uses both children. The **return value** to parent uses only one side (you can't go left and right in a single path from parent).

---

## Decision Guide

```
Need to process children before parent?
  └─ Postorder DFS

Need sorted order from BST?
  └─ Inorder DFS

Need level-by-level processing?
  └─ BFS with queue, snapshot size = q.Count before the loop

Need to compute something at every subtree (height, sum)?
  └─ Postorder — compute children first, combine at current node

Path passes through a node (diameter, max path sum)?
  └─ DFS returning height/gain, update global answer with l+r

Lowest common ancestor?
  └─ General tree: return node when found, split = LCA
  └─ BST: use ordering to navigate left/right

BST validation?
  └─ Pass min/max bounds — NEVER just compare to parent

Construct tree from traversals?
  └─ Preorder gives root, inorder splits left/right subtree
  └─ Use index map on inorder for O(1) lookup

AVOID recursion when:
  └─ Tree is very deep (stack overflow risk) → use iterative DFS with explicit stack
  └─ Need level info → BFS

AVOID DFS when:
  └─ Need shortest path root→node → BFS gives level directly
```

---

## Common Mistakes

1. **Null check placement** — always check `if (node == null)` first before accessing `node.left` or `node.val`.
2. **Min depth trap** — a node with one null child is not a leaf. Handle single-child nodes explicitly.
3. **BST validation with only parent check** — always pass `min`/`max` bounds, not just compare to parent.
4. **Path sum vs path gain** — in max path sum, a path through a node uses both children. What's returned to the parent uses only one. Mixing these up is the #1 bug.
5. **Forgetting to backtrack** in path sum III — after processing a node, decrement its prefix count before returning. Otherwise sibling subtrees see stale data.
6. **int overflow in path sum** — when summing node values across a path, use `long` if values can be large.

---

## Complexity Reference

| Operation | Time | Space |
|---|---|---|
| DFS traversal | O(n) | O(h) — h = height |
| BFS level order | O(n) | O(w) — w = max width |
| BST search/insert | O(h) | O(h) |
| Balanced BST h | O(log n) | — |
| Skewed BST h | O(n) | — |
| LCA | O(n) | O(h) |
| Build from traversals | O(n) | O(n) |

Space for DFS is O(h) recursion stack. Worst case O(n) for a skewed tree, O(log n) for a balanced tree.
