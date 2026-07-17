# Linked List — DSA Problem Solving Notes

## Core Definition

A linked list is a sequence of nodes where each node holds a value and a pointer to the next node. No random access — you must traverse from the head.

```csharp
class ListNode
{
    public int val;
    public ListNode next;
    public ListNode(int val) { this.val = val; }
}
```

**Key difference from array:**
```
Array:       random access O(1), insert/delete O(n)
Linked List: random access O(n), insert/delete O(1) given the node
```

---

## The Dummy Node Trick

Almost every linked list problem that modifies the list benefits from a dummy head. It eliminates special-casing the head node.

```csharp
var dummy = new ListNode(0);
dummy.next = head;
var curr = dummy;

// ... modify list ...

return dummy.next; // new head
```

---

## Core Patterns

### 1. Fast & Slow Pointers

Two pointers moving at different speeds. Used for cycle detection, finding midpoints, and Kth from end.

> LeetCode: [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/), [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/), [876. Middle of Linked List](https://leetcode.com/problems/middle-of-the-linked-list/), [19. Remove Nth from End](https://leetcode.com/problems/remove-nth-node-from-end-of-list/)

```csharp
// Middle of linked list — slow stops at mid when fast reaches end
ListNode MiddleNode(ListNode head)
{
    var slow = head;
    var fast = head;
    while (fast != null && fast.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow; // for even length, returns second middle
}
```

```csharp
// Remove Nth node from end — fast leads by n steps
ListNode RemoveNthFromEnd(ListNode head, int n)
{
    var dummy = new ListNode(0) { next = head };
    var slow = dummy;
    var fast = head;

    for (int i = 0; i < n; i++) fast = fast.next; // advance fast by n

    while (fast != null) { slow = slow.next; fast = fast.next; } // move together

    slow.next = slow.next.next; // remove the nth node
    return dummy.next;
}
```

**Why dummy here?** If the node to remove is the head (n == length), `slow` stays at dummy and `slow.next = slow.next.next` removes the head cleanly.

```csharp
// Detect cycle
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

---

### 2. Reverse a Linked List

Foundation for many problems. Master both iterative and recursive.

> LeetCode: [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/), [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/)

```csharp
// Iterative — three pointers
ListNode Reverse(ListNode head)
{
    ListNode prev = null;
    var curr = head;

    while (curr != null)
    {
        var next = curr.next; // save next
        curr.next = prev;     // reverse pointer
        prev = curr;          // advance prev
        curr = next;          // advance curr
    }
    return prev; // new head
}
```

**Trace: 1→2→3→4→null**
```
prev=null, curr=1
  next=2, 1.next=null, prev=1, curr=2
prev=1, curr=2
  next=3, 2.next=1, prev=2, curr=3
prev=2, curr=3
  next=4, 3.next=2, prev=3, curr=4
prev=3, curr=4
  next=null, 4.next=3, prev=4, curr=null

Result: 4→3→2→1→null ✓
```

```csharp
// Reverse between positions left and right (1-indexed)
ListNode ReverseBetween(ListNode head, int left, int right)
{
    var dummy = new ListNode(0) { next = head };
    var prev = dummy;

    for (int i = 1; i < left; i++) prev = prev.next; // walk to node before left

    var curr = prev.next;
    for (int i = 0; i < right - left; i++)
    {
        var next = curr.next;
        curr.next = next.next;   // detach next
        next.next = prev.next;   // insert next at front of reversed section
        prev.next = next;
    }
    return dummy.next;
}
```

**Trace: 1→2→3→4→5, left=2, right=4**
```
prev=1, curr=2

i=0: next=3, 2.next=4, 3.next=2, 1.next=3  → 1→3→2→4→5
i=1: next=4, 2.next=5, 4.next=3, 1.next=4  → 1→4→3→2→5

Result: 1→4→3→2→5 ✓
```

---

### 3. Merge Two Sorted Lists

> LeetCode: [21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/), [23. Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/)

```csharp
ListNode MergeTwoLists(ListNode l1, ListNode l2)
{
    var dummy = new ListNode(0);
    var curr = dummy;

    while (l1 != null && l2 != null)
    {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else                  { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = l1 ?? l2; // attach remaining
    return dummy.next;
}
```

---

### 4. Find Intersection of Two Lists

> LeetCode: [160. Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists/)

```csharp
// Two pointers — each walks both lists. They meet at intersection or null.
ListNode GetIntersectionNode(ListNode headA, ListNode headB)
{
    var a = headA;
    var b = headB;

    while (a != b)
    {
        a = a == null ? headB : a.next; // when A ends, switch to B
        b = b == null ? headA : b.next; // when B ends, switch to A
    }
    return a; // either intersection node or null
}
```

**Why it works:**
```
A: a1→a2→c1→c2→c3
B: b1→b2→b3→c1→c2→c3

Pointer a walks: a1,a2,c1,c2,c3,null → b1,b2,b3,c1
Pointer b walks: b1,b2,b3,c1,c2,c3,null → a1,a2,c1

Both arrive at c1 after traveling the same total distance (lenA + lenB).
```

---

### 5. Palindrome Linked List

Find middle → reverse second half → compare → restore (optional).

> LeetCode: [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/)

```csharp
bool IsPalindrome(ListNode head)
{
    // find middle
    var slow = head;
    var fast = head;
    while (fast != null && fast.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
    }

    // reverse second half
    var rev = Reverse(slow);
    var check = rev;

    // compare
    var curr = head;
    while (check != null)
    {
        if (curr.val != check.val) return false;
        curr = curr.next;
        check = check.next;
    }
    return true;
}

ListNode Reverse(ListNode head)
{
    ListNode prev = null;
    while (head != null)
    {
        var next = head.next;
        head.next = prev;
        prev = head;
        head = next;
    }
    return prev;
}
```

---

### 6. Reorder List

> LeetCode: [143. Reorder List](https://leetcode.com/problems/reorder-list/)

Pattern: find middle → reverse second half → merge two halves alternately.

```csharp
// L0→L1→L2→...→Ln  becomes  L0→Ln→L1→Ln-1→L2→Ln-2→...
void ReorderList(ListNode head)
{
    // 1. find middle
    var slow = head;
    var fast = head;
    while (fast.next != null && fast.next.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
    }

    // 2. reverse second half
    var second = Reverse(slow.next);
    slow.next = null; // cut the list

    // 3. merge alternately
    var first = head;
    while (second != null)
    {
        var tmp1 = first.next;
        var tmp2 = second.next;
        first.next = second;
        second.next = tmp1;
        first = tmp1;
        second = tmp2;
    }
}
```

**Trace: 1→2→3→4→5**
```
Middle: slow=3, cut → first=1→2→3, second=4→5
Reverse second: 5→4

Merge:
  first=1, second=5 → 1→5→2→4→3
```

---

### 7. Copy List with Random Pointer

> LeetCode: [138. Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)

```csharp
// HashMap: original node → cloned node
Node CopyRandomList(Node head)
{
    if (head == null) return null;
    var map = new Dictionary<Node, Node>();

    // first pass: create all clones
    var curr = head;
    while (curr != null) { map[curr] = new Node(curr.val); curr = curr.next; }

    // second pass: wire next and random
    curr = head;
    while (curr != null)
    {
        map[curr].next   = curr.next   == null ? null : map[curr.next];
        map[curr].random = curr.random == null ? null : map[curr.random];
        curr = curr.next;
    }
    return map[head];
}
```

---

### 8. Sort Linked List

> LeetCode: [148. Sort List](https://leetcode.com/problems/sort-list/)

Merge sort on linked lists — O(n log n) time, O(log n) space (recursion stack).

```csharp
ListNode SortList(ListNode head)
{
    if (head == null || head.next == null) return head;

    // find middle and split
    var slow = head;
    var fast = head.next; // start fast at head.next → slow lands at first half end
    while (fast != null && fast.next != null)
    {
        slow = slow.next;
        fast = fast.next.next;
    }
    var second = slow.next;
    slow.next = null; // cut

    var l = SortList(head);
    var r = SortList(second);
    return MergeTwoLists(l, r);
}
```

---

### 9. Add Two Numbers

> LeetCode: [2. Add Two Numbers](https://leetcode.com/problems/add-two-numbers/), [445. Add Two Numbers II](https://leetcode.com/problems/add-two-numbers-ii/)

```csharp
// Digits stored in reverse order (head = least significant)
ListNode AddTwoNumbers(ListNode l1, ListNode l2)
{
    var dummy = new ListNode(0);
    var curr = dummy;
    int carry = 0;

    while (l1 != null || l2 != null || carry > 0)
    {
        int sum = carry;
        if (l1 != null) { sum += l1.val; l1 = l1.next; }
        if (l2 != null) { sum += l2.val; l2 = l2.next; }
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
    }
    return dummy.next;
}
```

---

## Key Techniques Summary

```
Need to find middle?              → fast/slow — fast moves 2x
Need Kth from end?                → fast leads by k steps, then move together
Need to detect/find cycle?        → fast/slow — see TwoPointers notes
Modifying head might change?      → dummy node
Reversing a section?              → track prev, curr, next carefully
Problem on two lists?             → two pointer, one per list
Problem needs random access?      → copy to array first, solve, convert back
```

---

## Decision Guide

```
Find middle of list?
  └─ Fast/slow — slow at mid when fast hits end

Remove Nth from end?
  └─ Fast leads by N, move together, remove slow.next

Reverse entire list?
  └─ Iterative: prev/curr/next three pointers

Reverse a section?
  └─ Dummy + walk to left-1 + insert-at-front loop

Merge two sorted lists?
  └─ Dummy head, compare and advance

Palindrome check?
  └─ Find mid → reverse second half → compare

Sort a linked list?
  └─ Merge sort (find mid → split → sort each → merge)

Copy with random pointers?
  └─ HashMap: original → clone, two-pass

AVOID linked list when:
  └─ You need random access → convert to array first
  └─ You need binary search → array/BST
  └─ Problem involves indices → array cleaner
```

---

## Common Mistakes

1. **Lost node reference** — always save `curr.next` before changing `curr.next`. In reversal: `var next = curr.next` first.
2. **Null check for fast pointer** — always `fast != null && fast.next != null` before `fast.next.next`. Missing the first check causes NullReferenceException.
3. **Off-by-one in fast/slow for mid** — `fast = head.next` (not `head`) when you want slow at the first half's last node (for split). `fast = head` when you want slow at the true middle.
4. **Forgetting to cut the list** — after finding the middle for sort/palindrome, set `slow.next = null` to separate the two halves.
5. **Dummy node return** — always return `dummy.next`, not `dummy`.
6. **Carry after loop** — in add two numbers, the `while` condition must include `|| carry > 0` to handle a final carry digit.

---

## Complexity Reference

| Operation | Time | Space |
|---|---|---|
| Traversal | O(n) | O(1) |
| Reverse | O(n) | O(1) |
| Find middle | O(n) | O(1) |
| Merge two sorted | O(n+m) | O(1) |
| Sort (merge sort) | O(n log n) | O(log n) |
| Copy with random | O(n) | O(n) |
| Detect cycle | O(n) | O(1) |
| Find cycle entry | O(n) | O(1) |