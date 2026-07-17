# Heap — DSA Problem Solving Notes

## What is a Heap?

A heap is a complete binary tree where every parent satisfies the heap property:
- **Min-Heap** — parent ≤ children. Root = smallest element.
- **Max-Heap** — parent ≥ children. Root = largest element.

**Core intuition:** use a heap whenever you repeatedly need the min or max of a changing collection. It gives O(log n) insert/remove and O(1) peek.

```
Min-Heap example:
        1
       / \
      3   2
     / \ / \
    7  4 5  6

Root is always the minimum.
```

---

## C# Essentials

C# has `PriorityQueue<TElement, TPriority>` — a min-heap by default.

```csharp
// Min-Heap (default)
var minHeap = new PriorityQueue<int, int>();
minHeap.Enqueue(val, val);           // element, priority
minHeap.Peek();                      // peek min — O(1)
minHeap.Dequeue();                   // remove min — O(log n)
minHeap.Count;

// Max-Heap — negate the priority
var maxHeap = new PriorityQueue<int, int>();
maxHeap.Enqueue(val, -val);          // higher value = lower (negative) priority = dequeued first

// Custom priority (e.g. sort by frequency)
var heap = new PriorityQueue<string, int>();
heap.Enqueue("apple", freq["apple"]);
```

---

## Core Patterns

### 1. Kth Largest / Kth Smallest

**Kth Largest** → min-heap of size k. Root = kth largest.
**Kth Smallest** → max-heap of size k. Root = kth smallest.

> LeetCode: [215. Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/), [703. Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/)

```csharp
// Kth largest — keep a min-heap of size k
// When heap exceeds k, pop the smallest. Root = kth largest.
int FindKthLargest(int[] nums, int k)
{
    var heap = new PriorityQueue<int, int>(); // min-heap

    foreach (int n in nums)
    {
        heap.Enqueue(n, n);
        if (heap.Count > k) heap.Dequeue(); // remove smallest
    }
    return heap.Peek(); // root = kth largest
}
```

**Why min-heap for kth largest?**
```
We want to keep the k largest elements.
The min-heap root is the smallest of those k elements = kth largest overall.
When a new element arrives, if it's larger than the root, it kicks out the root.
```

```csharp
// Kth smallest — keep a max-heap of size k
int FindKthSmallest(int[] nums, int k)
{
    var heap = new PriorityQueue<int, int>(); // max-heap via negation

    foreach (int n in nums)
    {
        heap.Enqueue(n, -n);
        if (heap.Count > k) heap.Dequeue(); // remove largest
    }
    return heap.Peek();
}
```

---

### 2. Top K Frequent Elements

Build frequency map, then use a min-heap of size k keyed by frequency.

> LeetCode: [347. Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/), [692. Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/)

```csharp
int[] TopKFrequent(int[] nums, int k)
{
    var freq = new Dictionary<int, int>();
    foreach (int n in nums) freq[n] = freq.GetValueOrDefault(n, 0) + 1;

    var heap = new PriorityQueue<int, int>(); // min-heap by frequency

    foreach (var (val, count) in freq)
    {
        heap.Enqueue(val, count);
        if (heap.Count > k) heap.Dequeue(); // remove least frequent
    }

    var res = new int[k];
    for (int i = k - 1; i >= 0; i--) res[i] = heap.Dequeue();
    return res;
}
```

---

### 3. K Closest Points to Origin

> LeetCode: [973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/)

```csharp
// Max-heap of size k by distance. Root = kth closest.
// When heap exceeds k, pop the farthest.
int[][] KClosest(int[][] points, int k)
{
    var heap = new PriorityQueue<int[], int>(); // max-heap by distance

    foreach (var p in points)
    {
        int dist = p[0] * p[0] + p[1] * p[1]; // no need for sqrt
        heap.Enqueue(p, -dist);                 // negate for max-heap
        if (heap.Count > k) heap.Dequeue();
    }

    var res = new int[k][];
    for (int i = 0; i < k; i++) res[i] = heap.Dequeue();
    return res;
}
```

---

### 4. Merge K Sorted Lists / Arrays

Push the first element of each list into a min-heap. Always pop the smallest and push its successor.

> LeetCode: [23. Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/), [378. Kth Smallest in Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/)

```csharp
// Merge K sorted linked lists
ListNode MergeKLists(ListNode[] lists)
{
    // heap stores (node value, list index) — sorted by node value
    var heap = new PriorityQueue<ListNode, int>();

    foreach (var head in lists)
        if (head != null) heap.Enqueue(head, head.val);

    var dummy = new ListNode(0);
    var curr = dummy;

    while (heap.Count > 0)
    {
        var node = heap.Dequeue();
        curr.next = node;
        curr = curr.next;
        if (node.next != null) heap.Enqueue(node.next, node.next.val); // push successor
    }
    return dummy.next;
}
```

**Trace: lists = [[1,4,5],[1,3,4],[2,6]]**
```
Initial heap: [(1,L1),(1,L2),(2,L3)]

Pop 1 (L1) → push 4 (L1). heap: [(1,L2),(2,L3),(4,L1)]
Pop 1 (L2) → push 3 (L2). heap: [(2,L3),(3,L2),(4,L1)]
Pop 2 (L3) → push 6 (L3). heap: [(3,L2),(4,L1),(6,L3)]
Pop 3 (L2) → push 4 (L2). heap: [(4,L1),(4,L2),(6,L3)]
...

Result: 1→1→2→3→4→4→5→6 ✓
```

---

### 5. Median of a Data Stream (Two Heaps)

Maintain two heaps: max-heap for the lower half, min-heap for the upper half. Balance so sizes differ by at most 1.

> LeetCode: [295. Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/)

```csharp
class MedianFinder
{
    PriorityQueue<int, int> lo = new(); // max-heap — lower half
    PriorityQueue<int, int> hi = new(); // min-heap — upper half

    public void AddNum(int num)
    {
        lo.Enqueue(num, -num);  // always push to lo first

        // balance: lo's max must be <= hi's min
        if (hi.Count > 0 && lo.Peek() > hi.Peek())
        {
            int top = lo.Dequeue();
            hi.Enqueue(top, top);
        }

        // balance sizes: lo can have at most 1 more than hi
        if (lo.Count > hi.Count + 1)
        {
            int top = lo.Dequeue();
            hi.Enqueue(top, top);
        }
        else if (hi.Count > lo.Count)
        {
            int top = hi.Dequeue();
            lo.Enqueue(top, -top);
        }
    }

    public double FindMedian()
    {
        if (lo.Count > hi.Count) return lo.Peek();
        return (lo.Peek() + hi.Peek()) / 2.0;
    }
}
```

**Invariant:**
```
lo (max-heap) holds the smaller half
hi (min-heap) holds the larger half
lo.Count == hi.Count     → median = (lo.Peek() + hi.Peek()) / 2
lo.Count == hi.Count + 1 → median = lo.Peek()
```

---

### 6. Task Scheduling / Meeting Rooms

> LeetCode: [621. Task Scheduler](https://leetcode.com/problems/task-scheduler/), [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/)

```csharp
// Meeting Rooms II — min rooms needed for overlapping intervals
// Min-heap tracks end times of ongoing meetings
int MinMeetingRooms(int[][] intervals)
{
    Array.Sort(intervals, (a, b) => a[0].CompareTo(b[0])); // sort by start time

    var heap = new PriorityQueue<int, int>(); // min-heap of end times

    foreach (var interval in intervals)
    {
        if (heap.Count > 0 && heap.Peek() <= interval[0])
            heap.Dequeue(); // recycle room — meeting ended before new one starts
        heap.Enqueue(interval[1], interval[1]); // occupy room until end time
    }
    return heap.Count; // rooms still in use = min rooms needed
}
```

**Trace: [[0,30],[5,10],[15,20]]**
```
Sort: [[0,30],[5,10],[15,20]]

[0,30]:  heap empty → add. heap=[30]
[5,10]:  peek=30 > 5 → no room free → add. heap=[10,30]
[15,20]: peek=10 <= 15 → recycle → remove 10, add 20. heap=[20,30]

Result: heap.Count = 2 ✓
```

---

### 7. Sliding Window Maximum (Monotonic Deque)

Already covered in SlidingWindow notes. Use a deque, not a heap, for O(n) instead of O(n log n).

> LeetCode: [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/)

---

### 8. Reorganize / Greedy with Heap

Always pick the most frequent element available. Use a max-heap.

> LeetCode: [767. Reorganize String](https://leetcode.com/problems/reorganize-string/), [1405. Longest Happy String](https://leetcode.com/problems/longest-happy-string/)

```csharp
// Reorganize String — no two adjacent characters the same
string ReorganizeString(string s)
{
    var freq = new int[26];
    foreach (char c in s) freq[c - 'a']++;

    var heap = new PriorityQueue<char, int>(); // max-heap by frequency
    for (int i = 0; i < 26; i++)
        if (freq[i] > 0) heap.Enqueue((char)('a' + i), -freq[i]);

    var sb = new System.Text.StringBuilder();

    while (heap.Count > 0)
    {
        var (c1, p1) = (heap.Peek(), 0);
        heap.TryDequeue(out c1, out p1);
        sb.Append(c1);
        freq[c1 - 'a']--;

        // hold c1 aside, pick next most frequent
        if (heap.Count > 0)
        {
            var (c2, p2) = (heap.Peek(), 0);
            heap.TryDequeue(out c2, out p2);
            sb.Append(c2);
            freq[c2 - 'a']--;
            if (freq[c2 - 'a'] > 0) heap.Enqueue(c2, -freq[c2 - 'a']);
        }

        if (freq[c1 - 'a'] > 0) heap.Enqueue(c1, -freq[c1 - 'a']);
    }

    return sb.Length == s.Length ? sb.ToString() : "";
}
```

---

## Decision Guide

```
Need the min or max of a changing set repeatedly?
  └─ Heap — O(log n) insert/remove, O(1) peek

Kth largest?
  └─ Min-heap of size k. Root = answer.
  └─ AVOID sorting everything — O(n log k) beats O(n log n)

Kth smallest?
  └─ Max-heap of size k. Root = answer.

Top K frequent?
  └─ Frequency map + min-heap of size k by frequency

Merge K sorted sequences?
  └─ Min-heap with one element per sequence, push successor on pop

Running median?
  └─ Two heaps — max-heap (lower half) + min-heap (upper half)

Overlapping intervals, min rooms/resources?
  └─ Sort by start, min-heap of end times

Always want most/least frequent available?
  └─ Max/min heap, greedy pick

AVOID heap when:
  └─ Need Kth in static sorted array → binary search O(log n)
  └─ Need max of fixed window → monotonic deque O(n)
  └─ K is close to n → just sort O(n log n) is simpler
  └─ Need all elements sorted → sort directly, heap overhead not worth it
```

---

## Common Mistakes

1. **Max-heap via negation** — C# only has min-heap. Negate priority for max-heap: `heap.Enqueue(val, -val)`. Easy to forget.
2. **Heap size vs k** — for kth largest, pop when `Count > k` (after enqueue), not `Count >= k`.
3. **Two-heap balance order** — in median finder, always push to `lo` first, then rebalance. Reversing the order breaks the invariant.
4. **Merge K lists — pushing null** — always check `node.next != null` before pushing the successor.
5. **Using heap for sliding window max** — a heap gives O(n log n), a monotonic deque gives O(n). Don't reach for heap when window size is fixed.
6. **Comparing custom objects** — when heaping objects (points, intervals), make sure the priority captures the full comparison. Two points at equal distance need a tiebreaker or they'll overwrite each other.

---

## Complexity Reference

| Operation | Min-Heap | Notes |
|---|---|---|
| Peek min | O(1) | Root access |
| Insert | O(log n) | Bubble up |
| Remove min | O(log n) | Bubble down |
| Build from n elements | O(n) | Heapify |
| Kth largest (n elements) | O(n log k) | Better than O(n log n) sort |
| Merge K lists (n total) | O(n log k) | k = number of lists |
| Running median (n inserts) | O(n log n) | Two heaps |