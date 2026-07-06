# Stack & Queue — DSA Problem Solving Notes

## Core Definitions

**Stack** — LIFO (Last In First Out). Think: undo history, call stack, DFS.
**Queue** — FIFO (First In First Out). Think: print queue, BFS, task processing.
**Deque** — double-ended queue. Can push/pop from both ends. Covers both stack and queue.
**Monotonic Stack** — stack that maintains elements in sorted order by discarding elements that violate the order on push.

```csharp
var stack = new Stack<int>();
stack.Push(x);
stack.Pop();
stack.Peek();
stack.Count;

var queue = new Queue<int>();
queue.Enqueue(x);
queue.Dequeue();
queue.Peek();
queue.Count;

var deque = new LinkedList<int>();
deque.AddFirst(x);  deque.AddLast(x);
deque.RemoveFirst(); deque.RemoveLast();
deque.First.Value;   deque.Last.Value;
```

---

## Core Patterns

### 1. Valid Parentheses / Bracket Matching

Push opening brackets. On closing bracket, check top of stack matches.

> LeetCode: [20. Valid Parentheses](https://leetcode.com/problems/valid-parentheses/), [1249. Minimum Remove to Make Valid Parentheses](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/)

```csharp
bool IsValid(string s)
{
    var stack = new Stack<char>();
    var match = new Dictionary<char, char> { [')', '('], [']', '['], ['}', '{'] };

    foreach (char c in s)
    {
        if (!match.ContainsKey(c)) { stack.Push(c); continue; } // opening bracket
        if (stack.Count == 0 || stack.Peek() != match[c]) return false;
        stack.Pop();
    }
    return stack.Count == 0;
}
```

---

### 2. Monotonic Stack — Next Greater / Smaller Element

Maintain a stack of elements waiting for their answer. When a new element arrives, it resolves all smaller (or larger) elements in the stack.

> LeetCode: [496. Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/), [739. Daily Temperatures](https://leetcode.com/problems/daily-temperatures/), [84. Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/)

```csharp
// Next Greater Element — for each element, find the next larger one to the right
int[] NextGreaterElement(int[] nums)
{
    int n = nums.Length;
    var res = new int[n];
    Array.Fill(res, -1);
    var stack = new Stack<int>(); // stores indices, stack top = smallest unresolved

    for (int i = 0; i < n; i++)
    {
        // current element resolves all smaller elements waiting in stack
        while (stack.Count > 0 && nums[stack.Peek()] < nums[i])
            res[stack.Pop()] = nums[i];
        stack.Push(i);
    }
    return res;
}
```

```csharp
// Daily Temperatures — how many days until warmer temperature
int[] DailyTemperatures(int[] temps)
{
    int n = temps.Length;
    var res = new int[n];
    var stack = new Stack<int>(); // indices of unresolved days

    for (int i = 0; i < n; i++)
    {
        while (stack.Count > 0 && temps[stack.Peek()] < temps[i])
        {
            int j = stack.Pop();
            res[j] = i - j; // days waited
        }
        stack.Push(i);
    }
    return res;
}
```

**Trace: temps = [73, 74, 75, 71, 69, 72, 76, 73]**
```
i=0: stack=[0]
i=1: 74>73 → pop 0, res[0]=1. stack=[1]
i=2: 75>74 → pop 1, res[1]=1. stack=[2]
i=3: 71<75 → stack=[2,3]
i=4: 69<71 → stack=[2,3,4]
i=5: 72>69 → pop 4, res[4]=1
     72>71 → pop 3, res[3]=2
     72<75 → stack=[2,5]
i=6: 76>72 → pop 5, res[5]=1
     76>75 → pop 2, res[2]=4
     stack=[6]
i=7: 73<76 → stack=[6,7]

res = [1,1,4,2,1,1,0,0] ✓
```

---

### 3. Largest Rectangle in Histogram

Classic monotonic stack. Maintain indices of bars in increasing height order. When a shorter bar arrives, pop and compute area using it as the height.

> LeetCode: [84. Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/), [85. Maximal Rectangle](https://leetcode.com/problems/maximal-rectangle/)

```csharp
int LargestRectangleArea(int[] heights)
{
    var stack = new Stack<int>(); // indices, increasing height order
    int best = 0;
    int n = heights.Length;

    for (int i = 0; i <= n; i++)
    {
        int h = i == n ? 0 : heights[i]; // sentinel 0 at end flushes remaining stack

        while (stack.Count > 0 && heights[stack.Peek()] > h)
        {
            int height = heights[stack.Pop()];
            int width = stack.Count == 0 ? i : i - stack.Peek() - 1;
            best = Math.Max(best, height * width);
        }
        stack.Push(i);
    }
    return best;
}
```

**Trace: [2,1,5,6,2,3]**
```
i=0: stack=[0(h=2)]
i=1: h=1 < 2 → pop 0, width=1(stack empty→i=1), area=2×1=2. stack=[1(h=1)]
i=2: stack=[1,2]
i=3: stack=[1,2,3]
i=4: h=2 < 6 → pop 3, width=4-2-1=1, area=6×1=6
          2 < 5 → pop 2, width=4-1-1=2, area=5×2=10  ← best
          2 >= 1 → stop. stack=[1,4]
i=5: stack=[1,4,5]
i=6(sentinel): h=0
  pop 5(h=3), width=6-4-1=1, area=3
  pop 4(h=2), width=6-1-1=4, area=8
  pop 1(h=1), width=6(stack empty), area=6

Result: 10 ✓
```

---

### 4. Min Stack / Max Stack

Support push, pop, and getMin/getMax in O(1) by tracking the running min/max alongside each element.

> LeetCode: [155. Min Stack](https://leetcode.com/problems/min-stack/)

```csharp
class MinStack
{
    Stack<int> stack = new();
    Stack<int> minStack = new(); // tracks current min at each level

    public void Push(int val)
    {
        stack.Push(val);
        int currMin = minStack.Count == 0 ? val : Math.Min(val, minStack.Peek());
        minStack.Push(currMin);
    }

    public void Pop()
    {
        stack.Pop();
        minStack.Pop();
    }

    public int Top() => stack.Peek();
    public int GetMin() => minStack.Peek();
}
```

**Why parallel stack?**
When you pop the current min from the main stack, you need the previous min to be restored. Storing the min at each level handles this automatically.

---

### 5. Evaluate Expression / Calculator

> LeetCode: [227. Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/), [224. Basic Calculator](https://leetcode.com/problems/basic-calculator/)

```csharp
// Basic Calculator II — handles +, -, *, / (no parentheses)
int Calculate(string s)
{
    var stack = new Stack<int>();
    int num = 0;
    char op = '+'; // last seen operator

    for (int i = 0; i < s.Length; i++)
    {
        char c = s[i];
        if (char.IsDigit(c)) num = num * 10 + (c - '0');

        if ((!char.IsDigit(c) && c != ' ') || i == s.Length - 1)
        {
            if      (op == '+') stack.Push(num);
            else if (op == '-') stack.Push(-num);
            else if (op == '*') stack.Push(stack.Pop() * num);
            else if (op == '/') stack.Push(stack.Pop() / num);
            op = c;
            num = 0;
        }
    }

    return stack.Sum();
}
```

---

### 6. Implement Queue Using Stacks / Stack Using Queues

> LeetCode: [232. Implement Queue using Stacks](https://leetcode.com/problems/implement-queue-using-stacks/), [225. Implement Stack using Queues](https://leetcode.com/problems/implement-stack-using-queues/)

```csharp
// Queue using two stacks — amortized O(1) dequeue
class MyQueue
{
    Stack<int> inbox = new();   // for push
    Stack<int> outbox = new();  // for pop/peek

    public void Push(int x) => inbox.Push(x);

    public int Pop()
    {
        Refill();
        return outbox.Pop();
    }

    public int Peek()
    {
        Refill();
        return outbox.Peek();
    }

    void Refill()
    {
        if (outbox.Count == 0)
            while (inbox.Count > 0)
                outbox.Push(inbox.Pop()); // reverse order = FIFO
    }

    public bool Empty() => inbox.Count == 0 && outbox.Count == 0;
}
```

---

### 7. Decode String / Nested Structure

Stack naturally handles nesting — push context on open bracket, pop and merge on close.

> LeetCode: [394. Decode String](https://leetcode.com/problems/decode-string/), [726. Number of Atoms](https://leetcode.com/problems/number-of-atoms/)

```csharp
// Decode String — "3[a2[c]]" → "accaccacc"
string DecodeString(string s)
{
    var countStack = new Stack<int>();
    var strStack = new Stack<string>();
    var curr = new System.Text.StringBuilder();
    int k = 0;

    foreach (char c in s)
    {
        if (char.IsDigit(c))
        {
            k = k * 10 + (c - '0');
        }
        else if (c == '[')
        {
            countStack.Push(k);         // save repeat count
            strStack.Push(curr.ToString()); // save string built so far
            curr.Clear();
            k = 0;
        }
        else if (c == ']')
        {
            int repeat = countStack.Pop();
            string prev = strStack.Pop();
            var decoded = new System.Text.StringBuilder(prev);
            for (int i = 0; i < repeat; i++) decoded.Append(curr);
            curr = decoded;
        }
        else curr.Append(c);
    }
    return curr.ToString();
}
```

---

### 8. Monotonic Queue — Sliding Window Minimum

- A deque that maintains elements in increasing order. Front = current window minimum.
- A simpler solution is using priorityQueue
> LeetCode: [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/), [1438. Longest Subarray with Absolute Diff ≤ Limit](https://leetcode.com/problems/longest-continuous-subarray-with-absolute-diff-less-than-or-equal-to-limit/)

```csharp
// Sliding Window Minimum using monotonic deque
int[] SlidingWindowMin(int[] nums, int k)
{
    var dq = new LinkedList<int>(); // stores indices, front = min of window
    var res = new List<int>();

    for (int i = 0; i < nums.Length; i++)
    {
        if (dq.Count > 0 && dq.First.Value <= i - k)
            dq.RemoveFirst(); // out of window

        while (dq.Count > 0 && nums[dq.Last.Value] > nums[i])
            dq.RemoveLast(); // remove larger elements — they'll never be min

        dq.AddLast(i);
        if (i >= k - 1) res.Add(nums[dq.First.Value]);
    }
    return res.ToArray();
}
```

---

## Monotonic Stack — When to Use Which

```
Next Greater Element (to the right) → increasing stack (pop when current > top)
Next Smaller Element (to the right) → decreasing stack (pop when current < top)
Previous Greater Element            → increasing stack, process on push
Previous Smaller Element            → decreasing stack, process on push
Largest Rectangle / Trapped Water  → increasing stack, compute area on pop
```

---

## Decision Guide

```
Need LIFO order?
  └─ Stack

Need FIFO order?
  └─ Queue

Need both ends?
  └─ Deque (LinkedList in C#)

Nested structure (brackets, expressions)?
  └─ Stack — push on open, pop and merge on close

Next greater/smaller element for each position?
  └─ Monotonic stack — O(n), not O(n²)

Running min/max with push/pop?
  └─ Min/Max stack — parallel stack tracking min/max at each level

Fixed-size window min/max?
  └─ Monotonic deque — O(n)
  └─ AVOID heap here — O(n log k) is worse

Iterative DFS?
  └─ Stack (explicit call stack)

BFS level order?
  └─ Queue

AVOID stack when:
  └─ You need random access → use array/list
  └─ You need to search by value → use HashSet/Dictionary
  └─ Problem has no notion of order/nesting → stack adds no value
```

---

## Common Mistakes

1. **Empty stack check before Peek/Pop** — always check `stack.Count > 0` before accessing. Peeking an empty stack throws.
2. **Monotonic stack direction** — decide upfront: increasing (for next greater) or decreasing (for next smaller). Flipping it gives wrong answers silently.
3. **Width calculation in histogram** — `width = stack.Count == 0 ? i : i - stack.Peek() - 1`. The `-1` is needed because `stack.Peek()` is the last bar shorter than the popped bar, not the boundary itself.
4. **Forgetting the sentinel** — in histogram problems, append a `0` (or loop to `i <= n`) to flush remaining elements from the stack.
5. **Two-stack queue — wrong refill trigger** — only move from inbox to outbox when outbox is empty, not on every operation. Moving too early breaks FIFO order.
6. **Monotonic deque — remove expired front** — before adding to the result, check if the front index is outside `[i-k+1, i]` and remove it.

---

## Complexity Reference

| Operation | Stack | Queue | Deque | Monotonic Stack |
|---|---|---|---|---|
| Push / Enqueue | O(1) | O(1) | O(1) | O(1) amortized |
| Pop / Dequeue | O(1) | O(1) | O(1) | O(1) amortized |
| Peek | O(1) | O(1) | O(1) | O(1) |
| Next Greater (n elements) | — | — | — | O(n) total |
| Sliding Window Min/Max | — | — | O(n) | — |

Each element is pushed and popped at most once in a monotonic stack → O(n) total even though there's a while loop inside the for loop.